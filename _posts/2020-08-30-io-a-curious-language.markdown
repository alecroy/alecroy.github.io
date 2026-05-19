---
layout:     post
date:       2020-08-30 22:57:45 -0400 EDT
title:      "Io, a curious language"
categories: Software
---

I re-learned some Io this past weekend.

Io is one of the languages in that famous 7 Languages in 7 Weeks book.  I think the original edition, but maybe it was in the followup sequel.

Io is a prototype-object-oriented language runtime / interpreter.

It's a lot like Smalltalk, a little bit futuristic but also very simplistic.  It's not the most performant language, especially given its exceptionally lazy nature, but I have no reason to think it's especially slow.  I mean, it's garbage collected, if that's somehow a disqualifier.

It's a bit like Lua, in how few structures really seem to do it all, where other languages reach for special syntax sugar in all different ways, Io has almost no decoration to its form.

~~~io
foo bar(baz, quux, 
  zardoz := baz
  zardoz bar(quux, 3)
)
~~~

It's a little bit hard to read, at first, I guess.

It's certainly easier to read than Lisp.  It does feel like a *teeny* bit more syntax / sugar would help.  It's got lists, and everything's a hash, anyways, so what more do you need?

There is no object literal syntax, though(?), which is weird coming from ECMAScript.

I'm pretty sure there are *no* statements, which is weird to say, but pretty sure.  Assignment is a message, newlines are semicolons, semicolons are just terminators.

The terminology can be a bit confusing, as everything is so lazy, it's hard to talk about if something is a "call" or an "activation" or an "evaluation" &c.
