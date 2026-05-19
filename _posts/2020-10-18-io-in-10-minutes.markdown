---
layout:     post
date:       2020-10-18 10:50:00 -0400 EDT
title:      "Io in 10 minutes"
categories: Software
---

I try (not very well) to explain Io, a small language, a la `X in Y minutes`, as much as I know anyway

This is mostly a recollection from old notes, I wanted to get it all in one place (another place, perhaps).

[The real Io tutorial](https://iolanguage.org/tutorial.html) is at least as good, let's just be clear.  This is just a blog.

First, some wordy explainy parts, then below that `code` just like `X in Y minutes`.

## Io, in words

1. all values are __objects__, all actions are _messages_
1. __objects__ are prototypes, & classes, & namespaces, & contexts
1. ___blocks___ are functions, & methods, & closures
1. _messages_ are operators, & calls, & assigns, & accesses
1. __objects__ are:
   1. lists of slots, which are key/value pairs
   1. list of protos, which are __objects__ from which it inherits
1. __prototype objects__, usually Uppercase, have a `clone` slot
1. when an __object__ is cloned, its init slot is called
1. _messages_ will route to the __object's__ slots, then each of its __protos'__ slots until a matching slot is found.  This is fully dynamic and happens entirely at runtime.
1. Io has no globals.  The toplevel is an __object__ called the __Lobby__.
1. Slots can be created by assigning to them.

       Obj slot  :=  value
1. ___methods___ are anonymous functions that take a _message_ for a __receiver__, create a __locals object__, and assign the `proto` & `self` slot to the __receiver__.

       method("hello!" print)
1. ___methods___ return their last value by default
1. ___methods___ can take many arguments

       method(a, b, c, d, a*d - b*c)
1. ___blocks___ are basically methods, except their context is lexically-bound not to the __receiver/target__, but to the context where the block was created
1. for either ___blocks/methods___, its `call` slot holds information about the ___block/method___ activation: `sender`, `message`, `target`, `slotContext`, `activated`
1. an __object__ can both receive & pass a _message_ with `resend`.  This will forward the same _message_ to a __proto__.

## Io, in plain sight

I haven't double-checked all of this code, so I apologize if I've screwed anything up.

    #
    # Primitives, literals, declarations, assignments, I/O
    #
    true
    false
    nil
    1
    2.0
    1 + 2.0
    2 / 3.0
    "hello?"
    list(1, 2, "3")
                    # lists are heterogenous
    x := 1
                    # x is declared with :=
    x = 2
                    # x is updated with =
    "hello, world!" println
    "X = #{x}" interpolate println
                    # "X = 2"
    square := method(x, x*x)
    "#{x}^2 = #{square(x)}" interpolate println
                    # "2^2 = 4"
    ("pine" .. "apple") println
                    # "pineapple"
    (if(1 < 2, "1", "2") .. " is smaller") println
                    # if is an expression

    "X is #{x} before the loop" interpolate println
    while(x < 5, x = x + 1)
    "X is #{x} after the loop" interpolate println

    for(i, 1, 10, i print)
                    # 12345678910
    for(i, 1, 10, 3, i print)
                    # 14710

    fibonacci := method(n,
        if(n < 2,
            1,
            fibonacci(n - 2) + fibonacci(n - 1)
        )
    )

    fibonacciIterative := method(n,
        a := 0;
        b := 1;
        for(i, 1, n,
            next := a + b;
            a := b;
            b := next
        );
        b
    )

    Date cpuSecondsToRun(fibonacci(25)) println
    Date cpuSecondsToRun(fibonacciIterative(25)) println


    #
    # Objects & slots, in more detail
    #
    Object

    Object clone
            # different IDs
    Object clone

    Object sneeze := method("Achoo!" println)
    Object sneeze

    Object getSlot("sneeze")
            # get the method, don't invoke it
    Object getSlot("sneeze") code
            # ..yeah

    Lobby

    Lobby slotNames
    slotNames

    3 slotNames
    3 proto
    3 proto slotNames

    "A string" proto
    "A string" protos

    #
    # Message passing, in more detail
    #
    Knock := Object clone
    Knock whoIsIt     := method(call sender)
    Knock whoAmI      := method(call receiver)
    Knock whatIsIt    := method(call message name)
    Knock whatAboutIt := method(call message arguments)

    Knock whoIsIt
    Knock whoAmI
    Knock whatIsIt
    Knock whatAboutIt
    Knock whatAboutIt()
    Knock whatAboutIt(2, 3, x)

    Knock sayX := method(x println)

    SpecialContext := Object clone
    SpecialContext x := "special"

    Knock sayX
    SpecialContext Knock sayX
    SpecialContext Knock whatAboutIt(2, 3, x)

    #
    # Some "built-ins", provided you've got the JSON extension :P
    #   these probably won't work without it
    #
    list(1, 2, 3) println
    list(1, 2, 3) asJson println
    list(3, 1, 2) sort println
    list(1, 2, 3) sum println

    Config := Object clone
    Config version := "1.0"
    Config name := "io-language"
    Config asJson println

Again I'm not sure 100% of the above even "interprets", so to speak.

I would really like to expand a bit more, on maps, file/http I/O, coroutines.  And fix any mistakes.

Sometimes, I get a segfault running these examples through io, but I'm not really sure if it's user error or what.  I'll figure it out.
