---
layout:     post
date:       2021-01-16 09:34:11 -0500 EST
title:      "Boolean table programming"
categories: Software
---

Here's another take on if statements, though I'm quite partial to early return.  It's kind of a visual programming idea, since it's not real convenient to do this in plaintext.  At least, it's not convenient to parse.

1. Set up some booleans over existing values

2. A 2-d boolean table to nest each case

That's it.

You can't miss a case, you can't screw up your else-to-if matching, &c.

You probably shouldn't go combine more than 3 booleans, though.

In pseudo code,

    def fizzbuzz(n [Int])
      m3 = n % 3
      m5 = n % 5
      msg =
            m3          ~m3
          +-----------+-----------+
       m5 |"fizzbuzz" | "buzz"    |
          |           |           |
      ~m5 |"fizz"     | "$n"      |
          |           |           |
          +-----------+-----------+
      put msg

I hope I didn't screw up the pseudo code.

Anyhow, I hope the point comes across.  I guess fizzbuzz is the canonical thing to show here, and real code wouldn't apply as well.

I guess you could write statements, and whole code blocks in a corner/cell of the square/table, but it doesn't seem too crazy to just use the tables (and things in them) as values.

Why do we have to serialize everything into lines?

It's not very hard to write a table, especially if you oversize it.

### One boolean

      x              ~x
    +--------------+--------------+
    |              |              |
    +--------------+--------------+

### Two booleans

         x              ~x
       +--------------+--------------+
     y |              |              |
       +--------------+--------------+
    ~y |              |              |
       +--------------+--------------+

### Three booleans

Not sure how anyone would do this cleanly in 2-D, maybe I'm missing something..

           x              ~x
         +--------------+--------------+
     y z |              |              |
         |              |              |
      ~z |              |              |
         +--------------+--------------+
    ~y z |              |              |
         |              |              |
      ~z |              |              |
         +--------------+--------------+

### What else?

Well packing variable names is going to take up way too much space, so you've got to assign x = oneBoolean, y = theOther in a conventional way.

Let me just assume I have some table operator `#` or something

    def fizzbuzz(n [Int])
      m3 = n % 3
      m5 = n % 5
      msg = # m3 # m5
            m3          ~m3
          +-----------+-----------+
       m5 |"fizzbuzz" | "buzz"    |
          |           |           |
      ~m5 |"fizz"     | "$n"      |
          |           |           |
          +-----------+-----------+
      put msg

Or perhaps a sillier version, 3 boolean fizz buzz:

    def fizzbuzz(n [Int], silly [Bool])
      msg [Str] = # n % 3 # n % 5 # ~silly
             x              ~x
           +--------------+--------------+
       y z | "fizzbuzz"   | "buzz"       |
        ~z | "fuzzwuzz"   | "wuzz"       |
           +--------------+--------------+
      ~y z | "fizz"       | "$n"         |
        ~z | "fuzz"       | "$n!"        |
           +--------------+--------------+
      put msg

It's probably wise to type the whole table, to make sure you're not writing `Opt[Str, Int]` or something.  Maybe people wouldn't like tabling off of `~silly` and would flip the contents of the table, but I think certain things should be inverted.  Common case should come first.

I think I've seen a language that has something like this, but I can't recall the name, unfortunately ...
