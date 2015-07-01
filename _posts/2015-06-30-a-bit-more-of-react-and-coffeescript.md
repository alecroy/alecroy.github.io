---
layout: post
title: "A bit more of React & CoffeeScript"
date: 2015-06-30 23:32
---

I want to move past the hello, world example I posted yesterday and show something just a bit larger, the [React tutorial](https://facebook.github.io/react/docs/tutorial.html).  It's a pretty simple application demonstrating how to nest multiple components to build a simple Comments section.  If you've already done the React tutorial, there's nothing new to see here.  Without further ado, let's set up the basic groundwork then jump into the code.

## Setup

I'll use the same directory structure as before, though I'll be adding a few more files.

~~~bash
mkdir -p {build,src/{coffee,stylesheets},www}
touch src/coffee/Comment{,Box,Form,List}.coffee
touch src/coffee/app.coffee
touch www/index.html
touch comments.json
touch server.coffee
~~~

I'll use the same SASS build, and I'll need to download a couple extra JavaScript files, jQuery and Marked.

~~~bash
(cd src/stylesheets/ ; bourbon install ; neat install ; bitters install)
(cd www; wget http://necolas.github.io/normalize.css/latest/normalize.css)
(cd www; wget https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.2/marked.min.js)
(cd www; wget https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/jquery.min.js)
~~~

I'll start the same build pipeline, except I won't need the static HTTP server from before: the React tutorial uses a small dynamic server for posting new comments, which I'll get to below.

~~~bash
sass --watch src/stylesheets/style.sass:www/style.css &
coffee -o build/ -w -c src/coffee/* &
watchify -v -o www/bundle.js build/app.js &
~~~

## The code

Let's start with the HTML.  Not much has changed since hello, world; just add `<script>` includes for the jQuery and Marked files we downloaded earlier.

#### `www/index.html`

~~~html
<!DOCTYPE html>
<meta charset="utf-8">
<html>
  <head>
    <title>Comments</title>
    <link rel="stylesheet" type="text/css" href="normalize.css">
    <link rel="stylesheet" type="text/css" href="style.css">
  </head>
  <body>
    <script src="marked.min.js"></script>
    <script src="jquery.min.js"></script>
    <script src="bundle.js"></script>
  </body>
</html>
~~~

I'll reuse the same stub SASS file; you should be able to style this fairly easily given the class names we set up on the components later.

#### `src/stylesheets/style.sass`

~~~sass
@import 'bourbon/bourbon'
@import 'neat/neat'
@import 'base/base'
~~~

Next let's drop the example comments into a file `comments.json` at the top level of the project.  This won't be in `www/`.

#### `comments.json`

~~~json
[
    {
        "author": "Pete Hunt",
        "text": "This is one comment"
    },
    {
        "author": "Jordan Walke",
        "text": "This is *another* comment"
    }
]
~~~

The server is a pretty simple Express server.  We statically serve everything in `www/`, provide a `GET /comments.json` route to retrieve the list of comments, and provide a `POST /comments.json` route to accept individual comments and return the new list of comments.  This file sits next to `comments.json` in the top-level.

#### `server.coffee`

~~~coffeescript
fs = require 'fs'
path = require 'path'
express = require 'express'
bodyParser = require 'body-parser'

app = express()
app.use bodyParser.json()
app.use bodyParser.urlencoded extended: true

app.use '/', express.static path.join __dirname, 'www'

app.get '/comments.json', (req, res) ->
  fs.readFile 'comments.json', (err, data) ->
    res.setHeader 'Content-Type', 'application/json'
    res.send data

app.post '/comments.json', (req, res) ->
  fs.readFile 'comments.json', (err, data) ->
    comments = JSON.parse data
    comments.push req.body
    fs.writeFile 'comments.json', JSON.stringify(comments, null, 4), (err) ->
      res.setHeader 'Content-Type', 'application/json'
      res.setHeader 'Cache-Control', 'no-cache'
      res.send JSON.stringify comments

app.set 'port', process.env.PORT || 3000
app.listen app.get('port'), () ->
  console.log "Listening on :#{app.get 'port'}"
~~~

## The React code

Finally, we can talk about the React code!  There are 4 components and 1 top-level application file.

1.  Comment, which renders an individual comment
1.  CommentList, which renders a list of Comments from JSON
1.  CommentForm, an input form for posting new comments
1.  CommentBox, the outermost component holding a CommentList and the CommentForm

#### `src/coffee/Comment.coffee`

A Comment takes its author and its text content as properties and renders a small div.  It uses the `marked(..)` function from the Marked library included from the HTML.

~~~coffeescript
React = require 'react'
{div, h2, span} = React.DOM

module.exports = React.createFactory React.createClass
  displayName: 'Comment'
  render: ->
    rawComment = marked @props.children.toString(),
      sanitize: true
    div {className: 'comment'},
      h2({className: 'commentAuthor'}, @props.author)
      span(dangerouslySetInnerHTML: {__html: rawComment})
~~~

#### `src/coffee/CommentList.coffee`

CommentList is even simpler: it takes a JSON array as a property and renders a Comment for each element/comment in the array.

~~~coffeescript
React = require 'react'
{div} = React.DOM

Comment = require './Comment'

module.exports = React.createFactory React.createClass
  displayName: 'CommentList'
  render: ->
    div {className: 'commentList'},
      @props.data.map((comment) ->
        Comment({author: comment.author}, comment.text))
~~~

#### `src/coffee/CommentForm.coffee`

CommentForm takes a property that is a function, `onCommentSubmit`.  This function is provided by the CommentBox component and should handle updating the application state with the new comment; all CommentForm has to do is provide it with a single JSON comment.  After calling the handler, it clears out its input fields.

~~~coffeescript
React = require 'react'
{form, input} = React.DOM

module.exports = React.createFactory React.createClass
  displayName: 'CommentForm'
  handleSubmit: (e) ->
    e.preventDefault()
    author = React.findDOMNode(@refs.author).value.trim()
    text = React.findDOMNode(@refs.text).value.trim()
    if author and text
      @props.onCommentSubmit author: author, text: text
      React.findDOMNode(@refs.author).value = ''
      React.findDOMNode(@refs.text).value = ''
  render: ->
    form {className: 'commentForm', onSubmit: @handleSubmit},
      input({type: 'text', ref: 'author', placeholder: 'Your name'})
      input({type: 'text', ref: 'text', placeholder: 'Say something...'})
      input({type: 'submit', value: 'POST'})
~~~

#### `src/coffee/CommentBox.coffee`

This is the largest component by far.  It, too, takes some properties: the URL endpoint that points to the comments JSON and a polling interval for fetching new comments.  Polling isn't an ideal solution, but it's adequate for something this small; we'd need a more complicated `server.coffee` to push new comments out to clients, and it's mostly orthogonal to the concept of the application.

There are just 2 helper functions: `loadCommentsFromServer` is called internally to fetch new comments periodically, while `handleCommentSubmit` is passed down to a child component as a click/submit handler.  This is a pretty common pattern, keeping the state & the handler in the parent component, with the child component accessing the handler as a property.

~~~coffeescript
React = require 'react'
{div, h1} = React.DOM

CommentList = require './CommentList'
CommentForm = require './CommentForm'

module.exports = React.createFactory React.createClass
  displayName: 'CommentBox'
  loadCommentsFromServer: ->
    $.ajax
      url: @props.url
      dataType: 'json'
      cache: false
      success: (data) =>
        @setState data: data
      error: (xhr, status, err) =>
        console.error @props.url, status, err.toString()
  handleCommentSubmit: (comment) ->
    newComments = @state.data.concat [comment]
    @setState data: newComments
    $.ajax
      type: 'POST'
      data: comment
      url: @props.url
      dataType: 'json'
      success: (data) =>
        @setState data: data
      error: (xhr, status, err) =>
        console.error @props.url, status, err.toString()
  getInitialState: ->
    data: []
  componentDidMount: ->
    @loadCommentsFromServer()
    setInterval @loadCommentsFromServer, @props.pollInterval
  render: ->
    div {className: 'commentBox'},
      h1({}, 'Comments')
      CommentList({data: @state.data})
      CommentForm({onCommentSubmit: @handleCommentSubmit})
~~~

The top-level application file renders a CommentBox to `document.body` and passes the 2 properties, the URL for GET/POSTing the comments JSON, and the poll interval (in milliseconds).

#### `src/coffee/app.coffee`

~~~coffeescript
React = require 'react'
CommentBox = require './CommentBox'

React.render CommentBox(url: 'comments.json', pollInterval: 2000),
  document.body
~~~

## Finis

That's it!  At this point, you should be able to run the server and see a working comment box.  If you've already done the React tutorial, there's nothing new here except for the CoffeeScript.  Hopefully you see CoffeeScript is roughly as syntax-sugary as JSX, though there are obviously ups and downs to using either one.  Happy hacking.

~~~bash
coffee server.coffee 
Listening on :3000
~~~
