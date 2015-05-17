---
layout: post
title: "Understanding Metalsmith"
date: 2015-05-17 12:00
---

This past Saturday I learned a lot more about alternatives to Jekyll.  A local developer (thanks, Chris!) pointed me towards [assemble.io][asmbl], an all-Javascript solution.  That really piqued my interest, so I downloaded Assemble.  I think Jekyll is great, but adding a JS parser/generator tool to my toolbag would be worth it.

While I waited for an unusually lengthy `npm install` I stumbled upon [Hexo][hexo] and [Metalsmith][metal], among [others][st-gen].  I might stick to Assemble, but I'll admit it seemed like it might take a while to set up.  As long as I'm considering ditching Jekyll, why not shop around?

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

Diving into the source just to read it is not always a rewarding experience.  Not so! with Metalsmith.  I [cloned their repository][metal-gh] and looked at their `index.js`.  Like a lot of projects, it was just a thin wrapper over the `lib/` directory.

~~~javascript
module.exports = require('./lib');
~~~

So I opened `lib/` and found just one file, `index.js`.  *Surely something didn't get cloned*, I thought.  I opened it up and found just 344 lines.  *That's impossible, where is the code?*  I can read 344 lines of JavaScript.  That's not impossible.  So that's what I decided to do.

### The Source

Metalsmith has 2 interfaces: the CLI generator tool, and the JavaScript module.  I tended towards the JS side of things, but I'm no enemy of the command line.

#### The Library

`lib/index.js` has 14 required modules, 16 methods, and 1 constructor.  The constructor is written so you can instantiate it without `new`.

~~~javascript
var ms = Metalsmith(directory)...
~~~

I won't bore you with the required modules.  Of the 16 methods, 9 are simple getter/setters.

1.  `.clean(..)` touches `this._clean`
1.  `.concurrency(..)` touches `this._concurrency`
1.  `.destination(..)` touches `this._destination`
1.  `.directory(..)` touches `this._directory`
1.  `.frontmatter(..)` touches `this._frontmatter`
1.  `.ignore(..)` touches/appends to `this.ignores`
1.  `.metadata(..)` touches `this._metadata`
1.  `.source(..)` touches `this._source`
1.  `.use(..)` appends to `this.plugins`

That leaves 7 other methods.

1.  `.build()` builds files with the current settings and writes them into the destination directory
1.  `.path(filenames..)` resolves pathnames with respect to the root directory
1.  `.read(directory)` reads a directory (the source directory, by default) and returns a dictionary of files
1.  `.readFile(filename)` resolves a filename, reads it, and returns a hash `{ contents: <Buffer..> }`
1.  `.run(files, plugins)` runs `files` through `plugins` and returns a dictionary of files
1.  `.write(files, directory)` writes a dictionary of files to a directory (the source directory, by default)
1.  `.writeFile(filename, data)` resolves a filename and writes a data hash `{ contents: <Buffer..> }` to it

That's pretty much all there is to it.  There are still lots of middleware plugins for me to learn, but a pattern does seem to emerge.  It's nice when things are consistent.

~~~javascript
module.exports = plugin;

function plugin(opts){
  ...
  return function(files, metalsmith, done) {
    ...
      done();
    ...
  };
}
~~~

#### The Executable

Later I found some more code (just 163 lines) in `bin/metalsmith`.  Again, I'd rather do things in JS than use CLI tools by hand, but I took a look anyway.  It seemed a bit less complex than `lib/index.js`.  The executable does just a few things:

1.  Sets up usage information
1.  Sets up a help message
1.  Parses the command line arguments
1.  Parses the config file
1.  Instantiates a Metalsmith
1.  Loads the plugins, one by one
1.  Runs the build!

That's pretty much it.  It's got 3 helper functions and 4 required modules.  It uses `chalk` to color the terminal output, `commander` to make the CLI tool, and 2 methods from the native `fs` and `path` modules.  The helper functions, `fatal(msg, stack)`, `log(msg)`, and `normalize(obj)` aren't too lengthy.

## Finis

Hopefully I didn't get too much wrong.  I just found this tool today, so surely my views are subject to change.  I feel like I've got a much better handle on it now that I've read it.  Thanks to [Mike Valstar][valstar-home] for inspiring me to read the source,  and to [Segment][segment] for writing such a nice little piece of JavaScript.  Happy hacking.

[st-gen]: http://staticgen.com
[asmbl]: http://assemble.io
[hexo]: http://hexo.io
[metal]: http://metalsmith.io
[metal-gh]: https://github.com/segmentio/metalsmith/
[valstar]: http://mikevalstar.com/post/getting-started/
[valstar-home]: http://mikevalstar.com/
[segment]: https://segment.com/
