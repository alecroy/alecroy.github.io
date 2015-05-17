---
layout: post
title: "Understanding Metalsmith"
date: 2015-05-17 12:00
---

This past Saturday I learned a lot more about alternatives to Jekyll.  A local developer (thanks, Chris!) pointed me towards [assemble.io][asmbl], an all-Javascript solution.  That really piqued my interest, so I downloaded Assemble.  I think Jekyll is great, but adding a JS parser/generator tool to my toolbag would be worth it.

While I waited for an unusually lengthy `npm install` I stumbled upon [Hexo][hexo] and [Metalsmith][metal], among [others][st-gen].  I might stick to Assemble, but I'll admit it was a bit alarming to set up a few examples and see ~300MB of disk space gone.  As long as I'm considering ditching Jekyll, why not shop around?

## A challenger appears

Hexo stood out thanks to its easy quick start and great default theme.  I was generating and serving a blog in less than a minute.  Metalsmith actually didn't have any themed examples, but it just seemed so *simple*.  If you squint real hard, it looks like Express.

~~~javascript
var metalsmith = Metalsmith(__dirname)
  .use(concat)
  .build(function(err){
    if (err) throw err;
  });
~~~

That's from the `build.js` in examples/build-tool.  It appears to use middleware functions like `concat` in a pipeline.  The middleware style looks similar to Express: instead of `req` and `res` you operate on a hash of `files` and the `metalsmith` instance.  When your middleware has finished processing, you call `done`, just like `next`.

~~~javascript
function concat(files, metalsmith, done){
  ...

  files['index.css'] = {
    contents: new Buffer(css)
  };

  done();
}
~~~

The DIY theming seemed a bit spartan, but it does give you some freedom.  It's like a car that doesn't have doors: it's a lot easier to just get in and drive.

## Reading between the lines

I found a series of good blog [posts by Mike Valstar][valstar] discussing Metalsmith.  He mentioned looking at the source code as a good way to understand what it did.  I didn't feel like I *got it* from just reading the documentation, so I thought I'd give it a try.

Usually, diving into the source just to read it is not a rewarding experience.  I hate getting lost in 2000-line files.  I could not have been more wrong about Metalsmith.  I [cloned their repository][metal-gh] and looked at their `index.js`.  Like a lot of projects, it was just a thin wrapper over the `lib/` directory.

~~~javascript
module.exports = require('./lib');
~~~

So I opened `lib/` and found just one file, `index.js`.  *Surely something didn't get cloned*, I thought.  I opened it up and found just 344 lines.  *That's impossible, where is the code?*  I can read 344 lines of JavaScript.  That's not impossible.  So that's what I decided to do.

### The Source

Metalsmith has 2 interfaces: the CLI generator tool, and the JavaScript module.  I tended towards the JS side of things, but I'm no enemy of the command line.

#### The Library

#### The Executable

Later I found some more code (just 163 lines) in `bin/metalsmith`.  Again, I'd rather do things in JS than use CLI tools by hand, but I took a look anyway.  It seemed a bit less complex than `lib/index.js`.  The executable does just a few things:

1.  Sets up usage information
1.  Sets up a help message
1.  Parses the command line arguments
1.  Parses the config file
1.  Instantiates a Metalsmith
1.  Loads the plugins, one by one
1.  Runs the build!

That's pretty much it.  It's got 3 helper functions and 4 required modules.  It uses `chalk` to color the terminal output, `commander` to make the CLI tool, and 2 methods from the native `fs` and `path` modules.

[st-gen]: http://staticgen.com
[asmbl]: http://assemble.io
[hexo]: http://hexo.io
[metal]: http://metalsmith.io
[metal-gh]: https://github.com/segmentio/metalsmith/
[valstar]: http://mikevalstar.com/post/metalsmith-templating/
