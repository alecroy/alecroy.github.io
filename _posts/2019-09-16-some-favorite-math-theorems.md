---
layout: post
title: "Some Favorite Math Theorems"
date: 2019-09-16T20:58:40-04:00
categories: Math
params:
  math: true
---

These are just nice things to think about.  I don't know why, but some of them are just *nice*.
<!--more-->

I forget some of these, from time to time, so it's nice to write them all down in one place.  I may have made some mistakes paraphrasing them.  In no particular order:

## Goldbach's conjecture

> *every whole number can be written as the sum of two half-primes* (except 1)

Equivalently,

> *every whole number is the average of two primes* (except 1)

> *every even whole number can be written as the sum of two primes* (except 2)

## Legendre's conjecture

> \\(N^{2} .. (N+1)^{2}\\) contains a prime

## Opperman's conjecture

> \\(N^{2} ... N \cdot (N+1)\\) contains a prime, as does \\(N \cdot (N+1) ... (N+1)^{2}\\)

## Andrica's conjecture

> \\(\sqrt{p^{N}} < 1 + \sqrt{p^{N-1}}\\)

so, for example, \\(\sqrt{7} < 1 + \sqrt{5}\\), \\(\sqrt{11} < 1 + \sqrt{7} < 2 + \sqrt{5}\\), &c &c

## Firoozbahkt's conjecture

> \\(\sqrt[N+1]{p_{N+1}}  <  \sqrt[N]{p_{N}}\\)

so, for example, \\(\sqrt[4]{7} < \sqrt[3]{5} < \sqrt[2]{3}\\)

## Pollock's conjecture

Really going down a rabbit hole here

> *every whole number can be written as the sum of five tetrahedral numbers*

Tetrahedral numbers are physically the number of oranges you can stack in a pyramid with a triangular base, *N* oranges wide

$$ Te_{1} = 1 \\
Te_{2} = 1 + (1 + 2) = 4 \\
Te_{3} = 4 + (1 + 2 + 3) = 10 $$

The triangular layers form a sum of triangle numbers \\(T(N) = \frac{1}{2}  N  (N + 1)\\):

$$ Te_{N} = \sum_{i=1}^{N} T(N) $$

So if we have a bunch of triangles, and pair them with slightly smaller (or larger) triangles, the squares will complete, if you think about it:

> \\(T_{N-1} + T_{N} = 1^{2} + ... + N^{2}\\)

So they're each roughly half a sum-of-squares in maybe the same sense that triangular numbers are each roughly half of one square.

## Fermat's polygonal number theorem

Proven, though I wouldn't have expected it.  Fermat, Lagrange, Gauss, and Cauchy all solved some piece of it.

> *every whole number can be written as a sum of __n__ n-gonal numbers: 3 triangular numbers, 4 square numbers, 5 pentagonal numbers, ..*

..which of course raises all sorts of eyebrows and squints towards Goldbach's conjecture, as if half-primes were 2-sided/2-gonal numbers or something (?)

## Twin prime conjecture

> *there are infinitely many pairs of primes \\(N-1\\), \\(N+1\\)*

## Lehmer's totient problem

So \\(\phi(n)\\) is the *totient* of \\(n\\), the number of coprimes below \\(n\\).  This is \\(p-1\\) for a prime \\(p\\), and is more like \\(\pi(n)\\) for an \\(n\\) with many factors.

Known:

> *\\(\phi(p)\\) divides \\(p-1\\)*

and conjectured:

> *\\(\phi(c)\\) divides \\(c-1\\) for some composite \\(c\\)*

## Carmichael's totient problem

> *no whole number \\(n\\) has a unique totient \\(\phi(n)\\)*

In other words,

> *for every whole number \\(n\\), there is another, \\(m\\), such that \\(\phi(n) = \phi(m)\\)*

## Wolstenholme's problem

Known, supposedly:

> *if \\(n\\) is any prime number, \\(\dbinom{2n-1}{n-1} \equiv +1 \left( n^{3} \right)\\)*

what's conjectured is the other way around:

> *if \\(\dbinom{2n-1}{n-1} \equiv +1 \left( n^{3} \right)\\), then \\(n\\) must be prime*

## Collatz conjecture / Syracuse problem

In various flavors,

$$ c(n) = \begin{cases} 3n + 1 & \mathsf{n\ is\ odd} \\ \frac{1}{2}n & \mathsf{n\ is\ even} \end{cases} $$

I like to just count upwards to a power of 2, though:

$$ 15 \cdot 3^{5} + 2^{0} \cdot 3^{4} + 2^{1} \cdot 3^{3} + 2^{2} \cdot 3^{2} + 2^{3} \cdot 3^{1} + 2^{8} \cdot 3^{0} = 4,096 $$

where the idea is *any* odd number can be plugged in on the left as the leading coefficient of a \\(3^{x}\\) polynomial, along with other coefficients of \\(2^{y}\\) to all add up to *some* \\(2^{k}\\).

How to look at an odd number and *tell* how it will decompose (or rather, how the power of 2 will decompose into the polynomial), without going ahead and doing the algorithm ... well, that's the problem.

## Irrationals

A lot of these *could* go either way, I guess, but come on I think we all assume they're *at least* irrational:

$$ \zeta(5) \ \text{is irrational, as well as any \& all} \ \zeta(5+2k) $$

$$ \gamma \ \text{is irrational, algebraic, \&/ transcendental} $$

## abc conjecture

I vaguely kind of understand everything up to this point.  I *definitely* don't understand what's below.

This one's kind of a strange perspective: if we define some arbitrary "quality" to triplets of numbers, and only consider coprime triplets (no shared factors) as *q(a, b, c)*

> \\(q(a, b, c) = \dfrac{\log{c}}{\log{ \mathsf{radical}(a \cdot b \cdot c) }}\\)

where, bear with me, the "radical" of abc is just *the bottom row of primes, no exponents* if you express *a, b, c* each as prime products; in better words,

> \\(\mathsf{radical}(a \cdot b \cdot c) = \mathsf{the\ product\ of\ the\ distinct\ prime\ factors\ of\ n}\\)

So, we can finally say—err, conjecture—that, given the perspective that it's *rather hard* to exceed quality 1.0, but it can be done

> *for every quality \\(1+ε\\), there are only finitely many triples with \\(q(a, b, c) > 1+ε\\)*

Let's just focus around a rather fixed *c* value, and look at *a, b* pairs that could sum to roughly *c*.  This means, for a fixed *c*, we aim to minimize \\(\mathsf{radical}(a \cdot b \cdot c)\\).

...in other words, __it's rare to stumble upon two numbers *a, b* to be made of small prime powers that have a relatively large sum also made of small prime powers__ (different powers, all around), especially rare as we consider relatively larger and larger sums.

Even if you pick *a, b* to both be made of small primes, so as to minimize the *rad(abc)* on bottom (the *c* being mostly fixed), you still end up with enough big primes in c that it blows up the product.  If you do find a (relatively) large *c* that can in fact be made of small primes, you won't find many *a, b* pairs that are also made of small primes.

That's the idea, I think.  Just glancing around *c ≅ 125*:

> 128: \\(3 + 5^{3} = 2^{7}\\), \\(\dfrac{log(128)}{log{(2 \cdot 3 \cdot 5)}} = (q = 1.42)\\)


> 131: \\(2 \cdot 3 + 5^{3} = 131\\), \\(\dfrac{log(131)}{log{(2 \cdot 3 \cdot 5 \cdot 131)}} = (q = 0.59)\\)


> 132: \\(5^{3} + 7 = 2^{2} \cdot 3 \cdot 11\\), \\(\dfrac{log(132)}{log{(2 \cdot 3 \cdot 5 \cdot 7 \cdot 11)}} = (q = 0.63)\\)


> 133: \\(2^{3} + 5^{3} = 7 \cdot 19\\), \\(\dfrac{log(133)}{log{(2 \cdot 5 \cdot 7 \cdot 19)}} = (q = 0.68)\\)


So it is *rather unusual*, like we should expect.


## Birch-Swinnerton-Dyer conjecture

I'll have to come back around to this one; I remember reading about it.  It's a bit hard to grasp, even the concept.  There were a few minutes, maybe, where I understood it.  That time has passed.  Elliptic curves are kind of strange.

## Riemann's hypothesis

Ok, this is the one everybody knows and loves.  We all want a proof.  Let me just write out the badass mirror-image equation describing the complex extension, because who can remember that:

$$ \zeta(s) = \zeta(1 - s) · \Gamma(1 - s) · 2^{s} · \pi^{s-1} · sin(\frac{1}{2} \cdot s \cdot \pi) $$

where \\(\Gamma(1 - s)\\) is gamma, the complex extension of the factorial:

$$ \Gamma(x) = \int_{0}^{\infty} e^{-x} x^{z-1} \, dz $$

The simplest way I know to think about it is with the Mertens function \\(M(n)\\), which is "just" a running sum of the Mobius function[^0]:

$$ M(n) = \sum_{k=1}^{n} \mu_{k} $$

so the idea is we only care about the big-O of \\(M(n)\\):

$$ M(n) = O(n^{\frac{1}{2} + \epsilon})$$

Supposedly this is equivalent.  And the proof doesn't work without the \\(\epsilon\\).

Seems to me the bump in the rug is the Mobius function.  There's something going on with even-number-of-primes versus odd-number-of-primes, in a game where numbers-with-two-of-the-same-prime aren't even playing.  Strange.  No idea if anyone is ever going to solve it.

## Fermat's last theorem

Famously proven by Andrew Wiles[^3],

$$ A^{N} + B^{N} \neq C^{N}, N \geq 3 $$

Apparently, the connection to elliptic curves is that if we have some hypothetical solution \\(a\\), \\(b\\), \\(c\\), & \\(n\\) to the above, then we might want to care about the elliptic curve described by

$$ y^{2} = x \cdot (x - a^{n}) \cdot (x + b^{n}) $$

or, supposedly equivalently, (?)

$$ y^{2} = x \cdot (x - a^{n}) \cdot (x - c^{n}) $$

and as the story goes, this thing *can't* be modular (whatever *that* is..[^4]) if there really *is* some \\(a\\), \\(b\\), & \\(c\\).  But Wiles showed that *every elliptic curve*[^5] is modular, so that's that.  


__Note__: there have been so many errors in this post it's crazy, I've rewritten it over the years (as blog rerolls go) from MathJax to HTML to Markdown to Markdown+HTML back to MathJax that at this point I should just rewrite it from scratch.  I apologize for that.  It's a work in progress.

[^0]: which you just gotta *know*, for now, I can't explain it fast (or slowly)

[^3]: and not-quite-understood by almost every other human being on the planet, yours truly included

[^4]: don't ask me because I don't quite know

[^5]: talk about overkill
