---
title: "Shell scripting error handling (2)"
date: 2026-03-24T23:59:26Z
---

So zsh is basically compatible with bash.  But what about "proper" languages like Ruby, JavaScript, & Java?

Part of me kind of hates shell scripting, especially as you get into hundreds of lines, or thousands of lines.  I'm definitely of the mind to "just" write it in a real programming language.  Yet I still appreciate shell scripting for its exceptional & direct get-it-done nature.

## Ruby

Well, if you're using [Open3.popen3](https://docs.ruby-lang.org/en/master/Open3.html#method-c-popen3), you're passing in a single command, not a pipeline.  What you do with the `wait_thread.value` is up to you.

You can start up a pipeline with [Open3.pipeline_rw](https://docs.ruby-lang.org/en/master/Open3.html#method-i-pipeline_rw) and friends, but the logic is all up to you: you can `.join` each `wait_thread` and do with the `.value` what you wish.

So: maximum flexibility, but kind of low convenience, though.  You *don't need bash*, which can definitely be a strength.

The other options don't seem as great.  If you use backticks, you do get output, but don't get exit status.  If you use `system(...)`, you do get exit status, but you don't get output.  Hmm.

## JavaScript (zx)

Well, zx uses bash by default.  You can specify other shells

~~~
$.shell = '/bin/zsh'
...
zx --shell /bin/zsh script.js
~~~

but what about `-euxo pipefail` ?  

~~~
$.prefix

Specifies the command that will be prefixed to all commands run.

Default is set -euo pipefail;.
~~~

Well, they are just *that good*, I guess.  They think of everything.

## Java

Okay, so [jbang](https://www.jbang.dev) only really lets you *run* the Java, it doesn't help much in the way of subshells / pipelines / anything like that, as far as I can tell.

Enter ProcessBuilder.  Like Ruby, it's got a helper for starting up a pipeline: [ProcessBuilder.startPipeline(...)](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ProcessBuilder.html#startPipeline(java.util.List)).  You can get a [CompletableFuture<Process> from .onExit()](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Process.html#onExit()) to read the `.exitValue()` and do with what you want.

So it's basically the same interface as Ruby: maximum flexibility, but bash-less, so not convenient for one-liners.

## Overall

I think zx is the best solution I would still call "scripting".  I guess it's subjective, the line between a script and a program.

I guess I left out a lot of other languages, but I'm willing to bet they fall on the Ruby/Java side of things (which I'd describe as manual pipelining, with helpers).  I am curious if there are any close-equivalents to zx that build directly on bash.
