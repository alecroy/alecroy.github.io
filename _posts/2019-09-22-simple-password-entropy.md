---
layout: post
title: "Simple Password Entropy"
date: 2019-09-22T21:44:16-04:00
categories: Math
params:
  math: true
---

Passwords are easy to generate, easy to record.. but still nobody likes 45-character passwords!

I like to use [diceware](https://en.wikipedia.org/wiki/Diceware) to generate legible, secure passwords.  Generally, each word (out of 7,776) will add ~12–13 bits of entropy.  This requires 5 six-sided dice to be rolled for each word.
<!--more-->

6 words is *plenty*, good enough for just about anything.  4 words is enough for maybe your Netflix password[^0].

## Entropies

I'll use hand-rolled examples to show how weird the words can be.  Base64 sprinkled in liberally.  Don't use these passwords, obviously[^1].

| Pattern              | bits of entropy |
| -------------------- | --------------- |
| *MotocrossVariety-jj+G*
$2 \cdot log_2 (6^{5}) + 3 \cdot 8$ | 49.85 bits |
| *MotocrossVarietyGaveScroll*
$4 \cdot log_2 (6^{5})$ | 51.70 bits |
| *MotocrossVarietyGave-jj+G*
$3 \cdot log_2 (6^{5}) + 3 \cdot 8$ | 62.77 bits |
| *MotocrossVarietyGaveScrollFilter*
$5 \cdot log_2 (6^{5})$ | 64.62 bits |
| *MotocrossVariety-reckET89*
$2 \cdot log_2 (6^{5}) + 6 \cdot 8$ | 73.85 bits |
| *MotocrossVarietyGaveScroll-jj+G*
$4 \cdot log_2 (6^{5}) + 3 \cdot 8$ | 75.70 bits |
| *MotocrossVarietyGaveScrollFilterUncombed*
$6 \cdot log_2 (6^{5})$ | 77.55 bits |
| *MotocrossVarietyGave-reckET89*
$3 \cdot log_2 (6^{5}) + 6 \cdot 8$ | 86.77 bits |
| *MotocrossVarietyGave-MO+c1RK+jxdF*
$3 \cdot log_2 (6^{5}) + 9 \cdot 8$ | 110.77 bits |

So a little base64 goes a long way.  Too bad it's hard to write down accurately, let alone remember.

## Equivalent entropies

Some more common styles of passwords, for comparison (in increasing entropy):

| Pattern              | equation | bits of entropy |
| -------------------- | -------- | --------------- |
| *1234-56-7890* | $log_2 (10,000,000,000)$ | 33.22 bits |
| *wCEHMbIs* | $6 \cdot 8$ | 48 bits |
| *abcdefghijklm* | $13 \cdot log_2 (26)$ | 61.11 bits |
| *H65j/aS5vfmm* | $9 \cdot 8$ | 72 bits |
| *0mE07rdje4xzvxUE* | $12 \cdot 8$ | 96 bits |
| *aT7bubJTM4w2RoyeNPsQ* | $15 \cdot 8$ | 120 bits |

64 bits indeed seems "long".

A lot of bad password policies (6 chars only, 8 chars only..) keep you around 30-40 bits.

## Favorites

I like *MotocrossVarietyGaveScroll-jj+G* (75.70 bits) for long passwords, and *MotocrossVariety-jj+G* (49.85 bits) for shorter ones.  I like to be able to write them down more than I like them super short.

Generating these requires 2-4 dicerolls (& a diceware textfile lookup) and a quick `openssl rand -base64 3`.

64 bits is IMO "pretty good".  If you need 256 bits, you're not really talking about a *password* any more, but a *key*.  You can generate just lots more bits to come out of openssl, if that's what you're after

    $ openssl rand -base64 32
    bIxkkyQAh3GCxbiWurRpV5DsMdvcTTiTyMBWk2lpGcE=

## Update

I don't use a lot of long passwords, but looking back: *MotocrossVarietyGaveScroll-jj+G* (75.70 bits) is just super long.  31 characters!?  I wouldn't even try to remember those.

Why not go straight base64 at that point?  21 bytes gets you 28 characters.  18 bytes gets you 24 characters.  That's *144-168 bits* of entropy.  Way beyond normal stuff.  Here's a better "long" password:

__*If you can use a 20-character password, use 15 random bytes (120 bits) :*__

      $  openssl rand -base64 15
    aT7bubJTM4w2RoyeNPsQ     # results should vary..

This is way overkill, but a 20-character password is quite long and might as well be used.

You might need to add a special character (pick your favorite..) and *maybe* a number.  I guess, technically, you might land all `1`s or something crazy, so maybe append a `Az1!` or something to fulfill all those nice password requirements.  You may have to trim some bits to make it fit, if 24 characters is too many.

[^0]: typing, on those little remotes, is like maximum annoying

[^1]: but also don't avoid them, if you truly do roll them.. riiiight?
