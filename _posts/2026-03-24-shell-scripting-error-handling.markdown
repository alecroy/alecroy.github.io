---
title: "Shell scripting error handling"
date: 2026-03-24T02:30:45Z
---

If you already know about bash's `set -euo pipefail` and can explain every part individually, well, you're way ahead of me.

I've *known* about setting `-euo pipefail`, but I can't seem to recall what each part does.  Furthermore, I can't say what the equivalents are in other languages like Ruby, JavaScript/zx, Java/jbang, or even zsh!

So let's just review the basics, first ...

## set -euo pipefail, and friends

Let's take a look at what [mohanpedala wrote](https://gist.github.com/mohanpedala/1e2ff5661761d3abd0385e8223e16425) about it, who basically copied what [aaron maxwell wrote](http://redsymbol.net/articles/unofficial-bash-strict-mode/), though mohan's gist discussion also leads to an interesting [Cloudflare post](https://blog.cloudflare.com/pipefail-how-a-missing-shell-option-slowed-cloudflare-down/), so ups and downs.

`set -e` exits when any command completes with a non-zero exit status code.  I guess I wonder if bash *itself* exits with a non-zero code (and if it's the same code).  This is pretty strict, but reasonable.  You can temporarily disable it with `set +e ... set -e`.

`set -x` prints to the terminal the commands that are being run.

`set -u` exits when any undefined variable is referenced (instead of inserting like an empty string).

`set -o pipefail` raises any error in a pipeline to be the pipeline's exit code (and not just the final command in the pipeline).

Traps can run functions on exit, like `defer`s in Go.

~~~bash
function cleanup { ... }
trap cleanup EXIT
~~~

Okay, but what if I'm not using bash ?

## zsh

zsh is the new bash, at least as far as macOS is concerned.  The manual describes ...

~~~
$ man zshoptions
...
ERR_EXIT (-e, ksh: -e)
      If a command has a non-zero exit status, execute the ZERR trap,
      if set, and exit.  This is disabled while running initialization
      scripts.
...
~~~

It does look like there's a `PIPE_FAIL` option, but I can't tell if it has to be capitalized, or if the underscore is required, or what

~~~
PIPE_FAIL
      By default, when a pipeline exits the exit status recorded by
      the shell and returned by the shell variable $? reflects that of
      the rightmost element of a pipeline.  If this option is set, the
      exit status instead reflects the status of the rightmost element
      of the pipeline that was non-zero, or zero if all elements
      exited with zero status.
~~~              

`set -o pipefail` does seem to work the same way.


`set -u` seems to be the same

~~~
UNSET (+u, ksh: +u) <K> <S> <Z>
      Treat unset parameters as if they were empty when substituting,
      and as if they were zero when reading their values in arithmetic
      expansion and arithmetic commands.  Otherwise they are treated
      as an error.
~~~


`set -x` seems to dump a *ton* of debug information?  The docs seem to say it's pretty similar to bash:

~~~
...(bash)
  -x      After expanding each simple command, for command, case
          command, select command, or arithmetic for command,
          display the expanded value of PS4, followed by the
          command and its expanded arguments or associated word
          list.
...                      
...(zsh)
XTRACE (-x, ksh: -x)
      Print commands and their arguments as they are executed.  The
      output is preceded by the value of $PS4, formatted as described
      in the section EXPANSION OF PROMPT SEQUENCES in zshmisc(1).
...
~~~              

Hmm.  If I have a simple script like

~~~
% cat -n script 
     1	set -x
     2	uname
~~~

then it'll print like

~~~
% . ./script 
+./script:2> uname
Darwin
+update_terminal_cwd:5> local url_path=''                                       
+update_terminal_cwd:10> local i ch hexch LC_CTYPE=C LC_COLLATE=C LC_ALL='' LANG=''
+update_terminal_cwd:11> i = 1
...like 300 more lines of update_terminal_cwd
~~~

So it *does* seem to work as described, but it's not obvious where all this `update_terminal_cwd` stuff is coming from.  Interesting.

[It seems like it's a macOS thing](https://unix.stackexchange.com/a/748594) that can be worked around, so that's a wrap, I guess.  I can `set -euxo pipefail` to my heart's content in zsh as well.

