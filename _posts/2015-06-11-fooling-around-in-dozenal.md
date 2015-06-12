---
layout: post
title: "Fooling around in dozenal"
date: 2015-06-11 19:49
---

I wrote a couple small modules for fun recently and wanted to mention them here briefly.  I wrote one to calculate ranges of numbers and the other to convert numbers to dozenal, or duodecimal (base-12).

Now number ranges are hardly *new*, and maybe the code seems [Not Invented Here][nih].  But I didn't immediately stumble upon an existing solution that also calculated *logarithmic* number ranges (1, 10, 100, 1000, ..) so I wrote one up.

## var range = require('natural-number-range')

There are really only a few ways of using this module:

1.  `range(5)` gives you `[1, 2, 3, 4, 5]`
1.  `range(3, 1)` gives you `[3, 2, 1]`
1.  `range(1, 5, {step: 2})` gives you `[1, 3, 5]`
1.  `range(1, 1000, {scale: 10})` gives you `[1, 10, 100, 1000]`

That's it!  You can also use negative integers, but I think the *natural numbers* are the best use case, so I'm sticking with the name.

## var dozenal = require('dozenal')

This one's pretty much what you'd expect.  Numbers go in one side, strings come out the other.  Base-12 strings, that is.

`dozenal.print(12)` gives you `"10"`
`dozenal.print(10)` gives you `"T"`

That might throw you.  I didn't use the familiar letters `A`-`F` from hexadecimal to represent ten and eleven, but instead used `T` and `E`.

#### But wait, there's more!

I went ahead and did something really silly.  There's this feature in Common LISP's `format` function that *says* a number.  Let me just show you what I mean:

~~~lisp
* (format t "~r" 1048576)
one million forty-eight thousand five hundred seventy-six
NIL
~~~

I find this hilarious.  On some level, it's completely useless!  But it's just so darn *silly* they'd include that behavior, I went ahead and tried doing the same for base-12 numbers.  So lo and behold:

~~~javascript
> range(9, 12).map(function(x) { return [x, dozenal.say(x)] })
[ [ 9, 'nine' ], [ 10, 'dec' ], [ 11, 'el' ], [ 12, 'doh' ] ]
~~~

According to [this Numberphile video][numberphile] (my authoritative source on all things dozenal), you don't say "ten", "eleven", "twelve": those are all very *ten-centric* words.  *Tennist*, if you will.  Instead you say *dec*, *el*, and *doh*.  Ideally you'd say them with a British accent, but that's optional.

Going further, `100` is called a *gro*, like a single gross (= 12<sup>2</sup> = 144), and `1000` is a *mo* (= 12<sup>3</sup> = 1,728).

~~~javascript
> dozenal.say(1048576)
'four-gro two-doh six mo nine-gro nine-doh four'
~~~

In case you're dubious, this adds up:

-  'four-gro two-doh six mo' is a number of *mo*s
   - *four-gro* = 4 * 144 = 576
   - *two-doh* = 2 * 12 = 24
   - *six* = 6
   - for a total of (576 + 24 + 6) * 1,728 = 1,047,168
-  'nine-gro nine-doh four' is
   - *nine-gro* = 9 * 144 = 1,296
   - *nine-doh* = 9 * 12 = 108
   - *four* = 4
   - for a total of (1,296 + 108 + 4) = 1,408
- for a total of (1,047,168 + 1,408) = 1,048,576

I'll be honest: I kind of just made up the higher powers.  He goes up to a *gro* and a *mo* in the video, but past that I just made up names.  There *are* names out there, but nobody's really agreed upon any single set, so whatever.  Fork and pull request if it really bothers you.

One thing that did bother me was the way we label the higher powers of 10.  A *tri*llion is 10<sup>12</sup> and a *quad*rillion is 10<sup>15</sup>.  There's nothing remotely *quad*-ish about 10<sup>15</sup>.  15 isn't ever going to be divisible by 4.  And if you look at a trillion, it's actually got *four* groups of digits right of the leading 1.

~~~
1,000,000,000,000
~~~

So I broke the rules.  I say that a *tro* should have 3 groups of digits right of the leading 1.  And a *quadro* has 4 groups of digits.  A *mo* has just one group of digits, and a *bo* will have 2.  I mean, as long as we're fixing the base from 10 to 12, we might as well fix the powers, too.

~~~javascript
> range(1, 10000000000000, {scale: 12*12*12}).map(function(x) {
... return [x, dozenal.print(x), dozenal.say(x)]
... })
[ [ 1, '1', 'one' ],
  [ 1728, '1000', 'mo' ],
  [ 2985984, '1000000', 'bo' ],
  [ 5159780352, '1000000000', 'tro' ],
  [ 8916100448256, '1000000000000', 'quadro' ] ]
~~~

So a *quadro*, 12<sup>12</sup>, is actually just below nine trillion.  For your information.  Again, if you've got better names, fork & pull request!

## Finis

Thanks for reading!  Both of these packages are on npm ([natural-number-range][npm-nrr] & [dozenal][npm-d]), so `require` away!  Happy hacking.

[numberphile]: https://www.youtube.com/watch?v=U6xJfP7-HCc
[npm-nrr]: https://www.npmjs.com/package/natural-number-range
[npm-d]: https://www.npmjs.com/package/dozenal
[nih]: https://en.wikipedia.org/wiki/Not_invented_here
