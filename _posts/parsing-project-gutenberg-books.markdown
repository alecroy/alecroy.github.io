---
title: "Parsing Project Gutenberg Books"
date: 2026-03-30T23:51:57Z
draft: true
---

Project Gutenberg publishes a lot of free books.  They publish in a variety of formats, which is great.  I think the best format to parse is their HTML.  At least, that's my hope.

The goal is to basically split the HTML into chapters and put some styling on the result.  I'd like to try using CSS masonry on the paragraphs, to see how that looks from a typesetting point of view.

A small typesetting project, I guess.

I think, if I'm reading their license right, that I'll be deriving an ever-so-slightly new work, of perhaps lesser quality, so I should take any reference to Project Gutenberg out of the work itself.  I think I get what they're going for.

## Their structure

I can't speak for all of their books, but let's look at one

~~~
body
  section.pg-boilerplate.pgheader    // I think I can skip this
  div                                // for spacing
  table                              // INFO
  .fig                               // cover art
  h1                                 // The Title
  h2                                 // by Author
  hr
  h2                                 // Contents
  table
    tbody
      tr
        td                           // I.
        td
          a[href=#chapterN]          // Chapter One
      ...
  .chapter
    h2
      a#chapterN
      I.<br>Chapter One
    h3
      I.
    p.pfirst
      span.dropcap T
      ext of Chapter One ...
    p ...
~~~

One annoying thing is that some poems (maybe it's dialogue) will be set in `<pre>` tags, which doesn't really make sense to me.  I don't think that's something I can "just fix" with styling, so a full HTML -> HTML transform seems appropriate.


## What I'm thinking I want

I mean, something simple seems good enough.  I could get all semantic, I guess.  Each chapter could look like

~~~
body
  h1                                 // The Title
  h2                                 // by Author
  p ...
~~~

The table of contents could go a couple ways: either a numbered list of links as you'd usually see, or I could maybe give a little preview of each with its own first paragraph.  Or maybe both ...

## Fire up the parser

Let's go with node.  It seems like `parse5` is a good choice.

I kind of want to write something like XPath expressions to grab pieces of the tree.

~~~
/body/h1[0] -> The Title
/body/h2[0] -> by Author
/body/div.chapter[...] -> the chapters
~~~

I could do it imperatively, too, though.  