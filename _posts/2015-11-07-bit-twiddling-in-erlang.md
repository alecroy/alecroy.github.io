---
layout: post
title: "Bit twiddling in Erlang"
date: 2015-11-07 19:38
---

It's been a long few months.  Lots of traveling, interviewing, moving, & working.  Stressful.  I've been busy, to say the least.

I'm still learning Erlang.  It's one of my favorite languages to write.  I really like the Prolog-esque syntax, the interactive REPL, good [list processing](http://www.erlang.org/doc/man/lists.html), decent documentation, et cetera.  And I'm not even going to mention OTP (or Elixir).

One thing that stands out to me is the bitstring/binary processing, especially bit comprehensions.  It's much different from the masking & shifting I grew up with in C.  This perhaps isn't what Erlang is known for (coordination, availability, ..), but it's certainly not a weak point.

## Comprehensions

Lots of languages have a ```map``` function to apply a function to each element of a list.

~~~CoffeeScript
coffee> [1,2,3].map (n) -> -n
[ -1, -2, -3 ]
~~~

Erlang's got one: ```lists:map/2```.  Erlang's also got list comprehensions, too, which do roughly the same thing.

~~~Erlang
1> lists:map(fun(N) -> -N end, [1, 2, 3]).
[-1,-2,-3]
2> [ -N || N <- [1, 2, 3] ].
[-1,-2,-3]
~~~

So why use comprehensions?  For one, they're nice to read.  For two, ```lists:map/2``` will work on any list, but it won't work on bitstrings.  Bitstrings are like lists of bits, but you can't just go and use ```lists:map/2``` on a bitstring.  And there isn't a ```binary:map/2```, either!  If you want to map over a bitstring or binary, you'll probably want to use a comprehension (or [write your own ```binary:map/3```](http://chlorophil.blogspot.com/2007/06/erlang-binary-map.html)).

This sort of makes sense, though:  when you're mapping over a list, it's pretty clear how many things you want to map over at once: one (usually).  When you're mapping over a bitstring/binary, though, there's not really a good default: do you want to map over each bit?  Each byte?  Each word?

This is where bit comprehensions really shine:  you can pull off *any number* of bits at a time, and you can generate either bitstrings *or* lists.  Want to map over each bit?

~~~Erlang
1> Not = fun (<<Bit:1>>) -> 1 - Bit end.
#Fun<erl_eval.6.54118792>
2> Bits = <<240>>.
<<"ð">>
3> Bits_ = << <<(Not(<<B:1>>)):1>> || <<B:1>> <= <<Bits/bits>> >>.    
<<15>>
4> Bits = << <<(Not(<<B:1>>)):1>> || <<B:1>> <= <<Bits_/bits>> >>. 
<<"ð">>
~~~

Want to map over each bit, but generate a list?

~~~Erlang
1> [ Bit || <<Bit:1>> <= <<255>> ].
[1,1,1,1,1,1,1,1]
~~~

Want to break down a bitstring into 4-bit pieces?  Again, Erlang makes this pretty easy:

~~~Erlang
1> [ <<Nibble:4>> || <<Nibble:4>> <= <<15, 240>> ].
[<<0:4>>,<<15:4>>,<<15:4>>,<<0:4>>]
~~~

The only gotcha here is that if you *don't* annotate the width of the bitstring you're mapping to, it'll default to 8-bits wide:

~~~Erlang
1> FourBytes = << <<Nibble>> || <<Nibble:4>> <= <<15, 240>> >>.
<<0,15,15,0>>
2> byte_size(FourBytes).
4
3> FourNibbles = << <<Nibble:4>> || <<Nibble:4>> <= <<15, 240>> >>.
<<15,240>>
4> byte_size(FourNibbles).                                         
2
~~~

Isn't that nice?  I think so.  The syntax can be unfamiliar at first, but it's pretty clear what's going on.  Bitstring generators are a good way to use ```<=``` for something other than less-than-or-equals (like most languages do).
