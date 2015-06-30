---
layout: post
date: 2015-06-29 19:31
title: "A bit of React & CoffeeScript"
---

It's been a long two weeks; one large project and several interviews have just eaten up tons of my time.  I don't really like traveling; it really wears me out.

I just wanted to write a tiny bit of CoffeeScript today.  It's a bit refreshing how much code you don't have to write when you're writing CoffeeScript.  Nothing feels extraneous.  All killer no filler, as they say.  There are a couple good resources online for writing simple React stuff in CoffeeScript, but at least one of them is outdated, so I thought I'd rehash it.

## hello, world!

A real crowd pleaser, this one.  This is just to get you started.  First, let's create a basic directory structure to keep things separated.  Written source code goes in `src/` and generated source code goes in `build/`.  Inside `src/`, code is separated into `coffee/` (for CoffeeScript) and `stylesheets/` (for SASS).  I'm not going to write any styles today, but it's good to have it in the build pipeline.  Application/deliverable code goes in `www/`.

### A skeleton structure

~~~bash
mkdir -p {build,src/{coffee,stylesheets},www}

.
├── [  68 Jun 29 17:47]  build/
├── [ 272 Jun 29 17:47]  src/
│   ├── [  68 Jun 29 17:47]  coffee/
│   └── [  68 Jun 29 17:47]  stylesheets/
└── [  68 Jun 29 17:47]  www/
~~~

Now let's create our CoffeeScript & SASS source files, as well as our deliverable `index.html`.

~~~bash
touch src/coffee/app.coffee
touch src/stylesheets/style.sass
touch www/index.html

.
├── [  68 Jun 29 17:47]  build/
├── [ 340 Jun 29 17:47]  src/
│   ├── [ 102 Jun 29 17:49]  coffee/
│   │   └── [   0 Jun 29 17:52]  app.coffee
│   └── [ 102 Jun 29 17:49]  stylesheets/
│       └── [   0 Jun 29 17:52]  style.sass
└── [ 102 Jun 29 17:52]  www/
    └── [   0 Jun 29 17:52]  index.html
~~~

There's a very good starter set of SASS files we'll use: Bourbon, Neat, & Bitters.  You'll need to install these as Ruby Gems: `gem install bourbon neat bitters`.  Also, we'll install a common CSS reset: normalize.css.

~~~bash
(cd src/stylesheets; bourbon install)
(cd src/stylesheets; neat install)
(cd src/stylesheets; bitters install)
(cd www; wget http://necolas.github.io/normalize.css/latest/normalize.css)
~~~

### The build pipeline

Now let's start the build pipeline.  This will generate deliverable JavaScript and CSS from our sources (which are still empty).  You could set up a grunt/gulp/etc task to do this, but today I'll just do it on the command line.

If you don't have the CoffeeScript compiler installed, run a quick `npm install -g coffee-script`.  You'll also need to install Watchify to create your deployable Browserify bundle (`npm install -g watchify`), and any old HTTP server to view your work (`npm install -g http-server`).

First, run the CoffeeScript compiler to produce `build/app.js`.

~~~bash
coffee -o build/ -w -c src/coffee/* &

17:59:28 - compiled /.../src/coffee/app.coffee
~~~

Next, use that `build/app.js` to produce a Browserify bundle at `www/bundle.js`.

~~~bash
watchify -v -o www/bundle.js build/app.js &

564 bytes written to www/bundle.js (0.04 seconds)
~~~

Next, produce a CSS stylesheet at `www/style.css`.

~~~bash
sass --watch src/stylesheets/style.sass:www/style.css &

>>> Sass is watching for changes. Press Ctrl-C to stop.
      write www/style.css
      write www/style.css.map
~~~

Finally, start up an HTTP server to serve everything in `www/`.

~~~bash
http-server -p 8080 www/ &

Starting up http-server, serving www/ on: http://0.0.0.0:8080
Hit CTRL-C to stop the server
~~~

### Fill in the blanks

Now that we've got a live build pipeline, we can open up `http://localhost:8080/` and see our blank slate.  Let's fill in all three files.

#### `www/index.html`

~~~html
<!DOCTYPE html>
<meta charset="utf-8">
<html>
  <head>
    <title>hello, world!</title>
    <link rel="stylesheet" type="text/css" href="normalize.css">
    <link rel="stylesheet" type="text/css" href="style.css">
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
~~~

#### `src/stylesheets/style.sass`

~~~sass
@import 'bourbon/bourbon'
@import 'neat/neat'
@import 'base/base'
~~~

#### `src/coffee/app.coffee`

~~~coffeescript
React = require 'react'
{h3} = React.DOM

Message = React.createFactory React.createClass
  displayName: 'Message'
  render: ->
    h3 {}, "hello, #{@props.audience}!"

React.render Message(audience: 'world'), document.body
~~~

We're not using any JSX here; it's not exactly necessary given how nice CoffeeScript makes things.  We pull the `h3` function out of React.DOM, so instead of writing `<h3>hello, {this.props.audience}</h3>` we write `h3 {}, "hello, #{@props.audience}"`.  If you really miss JSX, you could probably find a compiler to work support writing that, but I'd rather not.

I create `Message` to just be a Factory that generates React Components; you could separate this into `message = React.createClass ..` and `Message = React.createFactory message` if you wanted to.  If you're ever confused what JavaScript we're outputting, check `build/app.js` to see some very readable output.  The `displayName` is something JSX will automatically set for debugging use; if you pop open the React tools in Chrome, that's what fills in the `<Message>..</Message>` tag name.

Down below in the `.render`, we write `Message(audience: 'world')` instead of `<Message audience="world" />`.  It's the same thing, only different.

## Finis

That's it!  Save all three files, watch the pipeline do its thing, and refresh your browser.  This wasn't supposed to be too complicated.
