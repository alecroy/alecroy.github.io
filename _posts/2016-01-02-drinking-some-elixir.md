---
layout: post
title: "Drinking some Elixir"
date: 2016-01-02 20:15
---

I got an Elixir book for Christmas, the one by Dave Thomas.  If you're anything like me (and I know I am), you get a little excited when someone puts a fresh twist on Erlang.

It's a bit different from Erlang, in a lot of ways.  Not in bad ways; you don't seem to lose much, and you do seem to gain quite a bit.  Here are just a few things I've noticed:

## Variables are lowercase, atoms have :colons

This is probably the first thing you get used to.  It's real minor.  I don't mind UppercaseVariables, but I don't mind lowercase ones, either!

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> Name = "Alec".
"Alec"
2> {ok, Name}.
{ok,"Alec"}
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> name = "Alec"
"Alec"
iex(2)> {:ok, name}
{:ok, "Alec"}
~~~

## Trailing commas are :ok

How can you not like these?  It's right in line with simplifying the somewhat overkill Erlang punctuation (so much ```;``` and ```.```!).

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> [1, 2, 3, ].
* 1: syntax error before: ']'
1>  
~~~
&#x1f641;

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> [1, 2, 3, ]
[1, 2, 3]
iex(2)>
~~~
&#x1f600;

## Perl-ish anonymous functions

Okay, it's not *just* Perl that's got these; Clojure has something very similar.  These help keep code short (and under 80 columns).

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> Enum.map([1, 2, 3], fn n -> n * n end)
[1, 4, 9]
iex(2)> Enum.map([1, 2, 3], &(&1 * &1))  
[1, 4, 9]
~~~

## Anonymous functions are called like ```.(...)```

This is a bit unintuitive, but it's in some way a feature.

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> (fun (N) -> N*N end)(5).
25
2> Square = fun (N) -> N*N end.
#Fun<erl_eval.6.54118792>
3> Square(5).
25
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> &(&1*&1).(5)
#Function<6.54118792/1 in :erl_eval.expr/5>
iex(2)> square = &(&1*&1)
#Function<6.54118792/1 in :erl_eval.expr/5>
iex(3)> square(5)
** (CompileError) iex:3: undefined function square/1

iex(3)> square.(5)
25
~~~

I say it's a feature because it actually distinguishes variables holding anonymous functions from named functions.  It's a bit like the difference between Lisp-1's and Lisp-2's: names of *variables holding anonymous functions* are in a separate namespace from *named functions*.  You don't have to worry about shadowing a named function with a variable.

Of course, it's the exact same thing in Erlang, but the capitalized variables make it impossible to mix up two different functions like ```Self``` and ```self```.  So the ```.(...)``` syntax lets you use lowercase variables without aliasing the already-lowercase functions.

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> self()  # self/0 is a built-in function
#PID<0.57.0>
iex(2)> self = fn -> "hello, this is computer" end                 
#Function<20.54118792/0 in :erl_eval.expr/5>
iex(3)> self.()  # call my anonymous version
"hello, this is computer"
iex(4)> self()   # call the built-in version
#PID<0.57.0>
~~~

Elixir does use UppercaseNames, but they're not for variables *or* functions (they're for Modules, Protocols, Records, Behaviours, ..).  (The book calls this BumpyCase -- honestly, what a silly name.  I've always heard it called CamelCase, but I guess [I've just](http://www.solipsys.co.uk/new/BumpyCaseWords.html) [been under](http://c2.com/cgi/wiki?BumpyCase) [a rock](https://en.wikipedia.org/wiki/CamelCase#Variations_and_synonyms).)

## Function literals are a bit different

I really do like that arity is more or less *part of* a function's name.  Elixir doesn't change that; it just replaces the ```fun``` out front with an ```&```.  (there's plenty of ```fun``` to go around...)

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> PassItHello = fun (Fn) -> Fn("hello\n") end.
#Fun<erl_eval.6.54118792>
2> PassItHello(fun io:put_chars/1).
hello
ok
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> pass_it_hello = fn fun -> fun.("hello\n") end
#Function<6.54118792/1 in :erl_eval.expr/5>
iex(2)> pass_it_hello.(&IO.write/1)
hello
:ok
~~~

## Bitstring syntax is a little different

Binaries (segments of bytes) are pretty much the same:

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> << 69, 114, 108, 97, 110, 103 >>.
<<"Erlang">>
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> << 69, 108, 105, 120, 105, 114 >>
"Elixir"
~~~

What's different is bitstrings (segments of non-8-bits-wide bitstrings):

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> << 6:4, 15:4, 6:4, 11:4 >>.
<<"ok">>
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> << 6::size(4), 15::size(4), 6::size(4), 11::size(4) >>
"ok"
~~~

Ok by me.

## Less than or equals to

I really don't mind how Erlang does this, but Elixir behaves more like, well, every other language I've ever used.

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> 3 =< 4.
true
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> 3 <= 4
true
iex(2)> 3 =< 4
** (SyntaxError) iex:2: syntax error before: '<'
~~~

## Paren-less function calls

Elixir is a bit like CoffeeScript-for-Erlang.  That analogy isn't perfect, but it's true.

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> max 3, 4
4
~~~

## Elixir modules are separate from Erlang modules

It's nice to know all the Erlang functions are still there for you to use.  The interop syntax is really convenient; you can't complain.

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> math:pi().
3.141592653589793
~~~

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> :math.pi()
3.141592653589793
~~~

## String interpolation

Pretty much CoffeeScript.

~~~Erlang
Eshell V7.2.1  (abort with ^G)
1> io_lib:format("The golden ratio is ~p~n", [(1+math:sqrt(5))/2]).
[84,104,101,32,103,111,108,100,101,110,32,114,97,116,105,
 111,32,105,115,32,"1.618033988749895","\n"]
2> lists:flatten(io_lib:format("The golden ratio is ~p~n", [(1+math:sqrt(5))/2])).
"The golden ratio is 1.618033988749895\n"
~~~

The flatten isn't really necessary; I just wanted a string for a proper comparison.  Erlang prints deep lists just fine, they're just weird to look at.

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> "The golden ratio is #{(1+:math.sqrt(5))/2}\n"
"The golden ratio is 1.618033988749895\n"
~~~

## Brace-less objects

Okay, these aren't *objects* per se, but really **keyword lists** (and they use *brackets*, not braces).  If you write them as literals, you need the brackets.  If you squint, this syntax is a *lot* like CoffeeScript.

Keyword lists *don't* need brackets if they're the last thing in a list of values (even if they're the only thing; it's where a list is *expected*).  Notably, this includes function calls & definitions.  They look like keyword arguments to the function, but they're actually just regular tuples in a list that's passed last.

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> benji = [{:name, "Benji"}, {:age, 5}]
[name: "Benji", age: 5]
iex(2)> phoebe = [name: "Phoebe", age: 9]
[name: "Phoebe", age: 9]
iex(3)> print_dog = fn dog -> "#{dog[:name]} is #{dog[:age]} years old" end
#Function<6.54118792/1 in :erl_eval.expr/5>
iex(4)> print_dog.( [name: "Benji", age: 5] )  # you can leave the brackets in
"Benji is 5 years old"
iex(5)> print_dog.( name: "Phoebe", age: 9 )   # or leave them out
"Phoebe is 9 years old"
~~~

There's nothing like this in Erlang; you would just write ```[{name, "Benji"}, {age, 5}]```, like on the first line.  Erlang is like black coffee when it comes to syntax sugar.

This works on function definitions, too; just specify multiple clauses in a single definition:

~~~Elixir
Interactive Elixir (1.2.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> greet = fn name, coming: true -> "hello, #{name}";
...(1)>            name, going: true -> "goodbye, #{name}";
...(1)>            name, coming: true, going: true -> "hi & bye, #{name}" end
#Function<12.54118792/2 in :erl_eval.expr/5>
iex(2)> greet.( "Alec", coming: true )
"hello, Alec"
iex(3)> greet.( "Alec", going: true )
"goodbye, Alec"
iex(4)> greet.( "Alec", coming: true, going: true )
"hi & bye, Alec"
~~~

## Finis

There's more to come, probably, but I want to get back to reading.  Happy hacking!
