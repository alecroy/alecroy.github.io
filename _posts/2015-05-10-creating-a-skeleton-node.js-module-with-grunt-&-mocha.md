---
layout: post
date:   2015-05-10 10:30
title:  "Creating a skeleton Node.js module with Grunt & Mocha"
---

It's just about every day now that I'm making Node modules from scratch.  Just to test ideas out, play around with some npm package, build projects, etc.  By the time you invent your fourth or fifth wheel, you stop reinventing from scratch and write it all down.  You could perhaps automate this into a single bash script, but there aren't *that* many steps to do by hand.  This will get you to a basic starting point so you can start writing unit tests & building a module.  By the end, your directory will look something like:

~~~bash
tree -a -L 2
.
├── .gitignore
├── .jscsrc
├── Gruntfile.js
├── module-name-here.js
├── node_modules
│   ├── .bin
│   ├── chai
│   ├── grunt
│   ├── grunt-contrib-jshint
│   ├── grunt-contrib-watch
│   ├── grunt-jscs
│   └── grunt-simple-mocha
├── package.json
└── test
    └── module-name-here_test.js

9 directories, 6 files
~~~

1.  Create a repository on GitHub where you'll be storing this work.  You don't *have* to publish it on GitHub; just keep it version-controlled.

1.  Get the repository on your machine.  I'll assume you put your repositories in a folder `~/GitHub/`, but it really doesn't matter.

    1.  Either clone a fresh copy of the repository,

        ~~~bash
        cd ~/GitHub/
        git clone https://github.com/USER/REPO.git
        cd ~/GitHub/REPO/
        ~~~

    1.  or link an existent repository to the one you created on GitHub,

        ~~~bash
        cd ~/path/to/REPO/
        git remote add origin https://github.com/USER/REPO.git
        git push -u origin master
        ~~~

    1.  or maybe you've already got a local copy sync'd with GitHub.

        ~~~bash
        cd ~/GitHub/REPO/
        git status
        On branch master
        Your branch is up-to-date with 'origin/master'.
        ~~~

1.  Create a subdirectory for your work. *(optional)*

    If this is just one module of many in the same repository, create a subdirectory and cd into it.

    ~~~bash
    mkdir SUBMODULE
    cd SUBMODULE/
    ~~~

1.  Set up your package file by running the `npm init` wizard.

    1.  Use a lowercase name: `module-name-here`, not `Module-Name-Here`.
    1.  Use version `0.0.1`.
    1.  Describe the module, briefly, so you don't forget what it does.
    1.  Use `module-name-here.js` as your entry point.  `index.js` is also an okay default.
    1.  Use `grunt test` as your test command.
    1.  Use `https://github.com/USER/REPO.git` as your git repository.
    1.  Enter a search keyword.  `example` or `exercise` are okay.
    1.  Enter your name.
    1.  Use the default `ISC` license.

1.  Install some Node.js modules.

    1.  If you don't already have `mocha` and `grunt-cli` installed globally, now would be the time to do it.  This only has to be done once for each machine you develop on.

        ~~~bash
        npm install -g mocha grunt-cli
        ~~~

    1.  Install some development dependencies locally.

        ~~~bash
        npm install --save-dev chai
        npm install --save-dev grunt
        npm install --save-dev grunt-contrib-jshint
        npm install --save-dev grunt-jscs
        npm install --save-dev grunt-simple-mocha
        npm install --save-dev grunt-contrib-watch
        ~~~

    1.  Add a one-line `.jscsrc` file.

        ~~~bash
        echo '{ "preset": "node-style-guide" }' >> .jscsrc
        ~~~

1.  Add a one-line module file `module-name-here.js`.

    ~~~bash
    echo "'use strict';" >> module-name-here.js
    ~~~

1.  Add a test folder with a stub test file `test/module-name-here_test.js` that contains:

    ~~~javascript
    'use strict';

    var moduleNameHere = require('../module-name-here');
    var expect = require('chai').expect;

    expect(moduleNameHere).to.not.equal(null);
    ~~~

1.  Set up your Gruntfile.

    This might look like a lot but once it's configured you can mostly set it and forget it.  Just `touch Gruntfile.js` and open it in an editor to contain:

    ~~~javascript
    'use strict';

    module.exports = function(grunt) {
      grunt.initConfig({
        jshint: {
          all: {
            src: ['./*.js', './test/**/*_test.js'],
            options: {
              curly: true,
              eqeqeq: true,
              expr: true,
              mocha: true,
              node: true,
              strict: true,
              undef: true,
              unused: true,
            },
          },
        },
        jscs: {
          all: {
            src: ['./*.js', './test/**/*_test.js'],
            options: {
              config: '.jscsrc',
              verbose: true,
            },
          },
        },
        simplemocha: {
          all: {
            src: ['./test/**/*_test.js'],
          },
        },
        watch: {
          files: ['./*.js', './test/**/*_test.js'],
          tasks: ['lint', 'test'],
        },
      });

      grunt.loadNpmTasks('grunt-contrib-jshint');
      grunt.loadNpmTasks('grunt-simple-mocha');
      grunt.loadNpmTasks('grunt-contrib-watch');
      grunt.loadNpmTasks('grunt-jscs');

      grunt.registerTask('test', 'simplemocha');
      grunt.registerTask('lint', ['jshint', 'jscs']);

      grunt.registerTask('default', ['lint', 'test']);
    };
    ~~~

    Probably the most confusing part of that is why some tasks are configured with `{ all: { src: .. , options: ..}}` and others are configured with `{ files: .. , .. }`.  The reason is that standard Grunt tasks do one thing and one thing only.  In this case, `watch` is a standard task, and there is only one kind of watch to run: `grunt watch`.  The others are *multitasks*, which means you can set up multiple versions of the same task.  For example, I could split apart my `jshint` task into two: one for the source files, one for the test files.  It would look like:

    ~~~javascript
    jshint: {
      sources: {
        src: ['./*.js'],
        options: {
          ...
        },
      },
      tests: {
        src: ['./test/**/*_test.js'],
        options: {
          ...
        },
      },
    },
    ~~~

    Now I would have two grunt tasks I can run: `grunt jshint:sources` and `grunt jshint:tests`.

1.  Run `grunt`.  If all goes well, you'll see something like:

    ~~~bash
    Running "jshint:all" (jshint) task
    >> 3 files lint free.

    Running "jscs:all" (jscs) task
    >> 3 files without code style errors.

    Running "simplemocha:all" (simplemocha) task


      0 passing (1ms)


    Done, without errors.
    ~~~

    Instead of running `grunt` by hand every other minute, you can run `grunt watch`, which will automatically re-run the `lint` and `test` tasks every time a JavaScript file changes.  You could have it watch for other files, too (like test data sets); just edit that part of the Gruntfile:

    ~~~javascript
    watch: {
      files: ['./*.js', './test/**/*_test.js', 'more patterns..'],
      tasks: ['lint', 'test'],
    },
    ~~~

1.  Set up your git ignore file. *(optional)*

    If you try committing all these changes right away, git might try adding all those packages we installed earlier (grunt, chai, ..).  If you're very careful with what you add to the index, you can skip this step.  If you like to run `git add .`, you should not skip this step.  This is what it looks like when git isn't ignoring your packages:

    ~~~bash
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Untracked files:
      (use "git add <file>..." to include in what will be committed)

            ...
            node_modules/
            ...
    ~~~

    It doesn't make much sense to bloat your repository with packages that are always available over the Internet.  Instead, we'll tell git to ignore those files:

    ~~~bash
    echo 'node_modules' >> .gitignore
    ~~~

1.  Commit all the changes to GitHub.

    ~~~bash
    git add .
    git commit
    git push
    ~~~

1.  That's it!  Coffee time.  Happy hacking.
