---
layout: post
date:   2015-05-09 08:03
title:  "Grunt, from a newcomer's perspective"
---

## The Grunt tool

I've used Makefiles in the past, but you can't write Makefiles in JavaScript.  I'd rather just write JavaScript if I could; one language is simpler than two.

Make is *the* straightforward build tool.  Grunt is not a make replacement, but it sure is useful.

Probably the strangest part is that grunt is split into 2 parts:

1.  the command line interface wrapper, installed globally: _grunt-cli_
1.  the task runner itself, installed locally: _grunt_

There is at least one advantage to this:  different projects can rely on different versions of grunt without breaking.  By installing specific versions of the task runner locally for each project, we don't need one super-compatible version installed system-wide.

In contrast, the behavior of make is not going to change.  It can grow, but it will (most likely) remain backwards-compatible, running old Makefiles without a hitch.

The grunt CLI wrapper doesn't need to do very much at all, then, except locate the local copy of grunt.  36 sloc, [more or less](https://github.com/gruntjs/grunt-cli/blob/master/bin/grunt).  Brilliant.


## The Gruntfile

Make uses a Makefile.  Grunt uses a Gruntfile.  It's a JavaScript file, so it's literal filename is `Gruntfile.js`.  Here's the most basic Gruntfile you can write; it does absolutely nothing.

{% highlight javascript %}
'use strict';

module.exports = function(grunt) {
  grunt.initConfig({
  });

  grunt.registerTask('default', []);
};
{% endhighlight %}

Running `grunt` on this Gruntfile should just say `Done, without errors.`.

Plugins should be configured in the call to `.initConfig(..)`, then loaded just below it as `.loadNpmTasks('plugin-name');`.  For example, `simplemocha` is a [plugin available through npm](https://www.npmjs.com/package/grunt-simple-mocha) that can be set up like so:

{% highlight javascript %}
'use strict';

grunt.initConfig({
  simplemocha: {
    all: {
      src: ['./test/**/*_test.js']
    }
  },
});

grunt.loadNpmTasks('grunt-simple-mocha');
{% endhighlight %}

This sets up a `simplemocha:all` task to run `mocha` on all the `.._test.js` files in the `test/` directory.  Clearly this isn't the *target / dependencies / build step* layout that Makefiles tend towards, but it's not far off.  It's the plugins that make grunt shine; each plugin operates in its own way on its own source files.  The [watch plugin](https://www.npmjs.com/package/grunt-contrib-watch), for example, is especially useful for interactive development & testing, rerunning lint/test routines on every file change.  Next post, I'll get into a more complex Gruntfile.  For now, happy hacking.
