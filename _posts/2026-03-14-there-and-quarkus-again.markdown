---
title: "There and Quarkus Again"
date: 2026-03-14T00:03:53Z
---

It's been a while since I looked at Quarkus.  I remember reading through the little free O'Reilly book on vert.x when that came out years ago, and I really liked it, and I noticed at some point Quarkus was built on it, but I guess I lost track ...

## There's a lot going for it

1. really good documentation

1. really great CLI for scaffolding / build tasks

1. deployment flexibility, from normal jars to Docker images to native ...

1. gRPC works basically out of the box

1. Kubernetes works basically out of the box

I mean, unless you really hate Java, it's ... not bad.

## Always some downsides

I like Ruby, & Rails.  Who doesn't?  A lot of people, I guess.  I've read a lot of criticism of Rails having to do with like "magic" or "variables from nowhere" or however you want to put it.  Basically, an abundance of *implicit* wiring up of how things interoperate.

@Lots @Of @Annotations, yes.  It's a pretty reasonable way to write DI, though.

If you thought Spring or Hibernate had a lot of annotations, well, I don't know what to tell you.  The annotations basically form an entire language unto itself.  It's just how it is.

Like the math guy said, you can get used to just about anything.

There's perhaps a *ton* of it in Quarkus, but again, it's ... not bad.

It's really not that far off from Lisp macros, if you squint at it a bit.  Code generation has upsides!

## Where to, from here ?

So what are they working on now?  Looks like:

1. HTTP/3

1. smaller container builds (hello world is like 500 MB, so ...)

1. upgrades to upstream: Vert.x, Netty, Jackson ...

All in all ... not bad.

