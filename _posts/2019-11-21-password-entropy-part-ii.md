---
layout: post
title: "Password Entropy, Part II"
date: 2019-11-21T15:56:54-05:00
categories: Math
params:
  math: true
---

Passwords are, generally speaking, expensive to crack.

> *Even assuming a centralized effort could be an order of magnitude more efficient, this still leaves us with an estimate of US\\$1M to perform a $2^{70}$ SHA-256 evaluations and around US\\$1B for $2^{80}$ evaluations[^0].*

So it would cost \\$1M to crack a 71-bits-of-entropy password (on average..), and \\$1B to crack an 81-bit password.

| Pattern              | bits of entropy | cost to crack |
| -------------------- | --------------- | ------------- |
| *MotocrossVarietyGaveScroll*
$4 \cdot log_{2} (6^{5})$ | 51.70 bits | <\\$1M |
| *MotocrossVarietyGaveScrollFilter*
$5 \cdot log_{2} (6^{5})$ | 64.62 bits | <\\$1M |
| *MotocrossVarietyGaveScrollFilterUncombed*
$6 \cdot log_{2} (6^{5})$ | 77.55 bits | \\$1M - \\$1B |

and the non-diceware passwords:

| Pattern              | equation | bits of entropy | cost to crack |
| -------------------- | -------- | --------------- | ------------- |
| *1234-56-7890* | $log_2 (10,000,000,000)$ | 33.22 bits | probably a buck |
| *wCEHMbIs* | $6 \cdot 8$ | 48 bits | could probably do it on an iPad |
| *abcdefghijklm* | $13 \cdot log_2 (26)$ | 61.11 bits | <\\$1M |
| *H65j/aS5vfmm* | $9 \cdot 8$ | 72 bits | \\$1M |
| *0mE07rdje4xzvxUE* | $12 \cdot 8$ | 96 bits | more than \\$1B |
| *aT7bubJTM4w2RoyeNPsQ* | $15 \cdot 8$ | 120 bits | way more than \\$1B |

Now this is all Assuming lots of things, like:

  -  the hash algorithm is SHA256, and not SHA224 or SHA512 or SHA3 or scrypt

  -  we're just doing one round (I think), not *sha256(..sha256(x)..)*

  -  we can ignore all(?) of the known attacks on SHA256, like collision attacks (finding the collision would be even harder?), length extension attacks (again, harder?), attacks we don't even know about...

  -  the source is right

They provide a source paper[^1] that goes into depth:

> *In 2013, Bitcoin miners collectively performed ≈ $2^{75}$ SHA-256 hashes in exchange for bitcoin rewards worth ≈ US\\$257M.  ...  Even assuming a centralized effort could be an order of magnitude more efficient, this still leaves us with an estimate of US\\$1M to perform a $2^{70}$ SHA-256 evaluations and around US\\$1B for $2^{80}$ evaluations.*

[^0]: [Lobste.rs discussion](https://lobste.rs/s/x6bt1h/xkcd_s_correcthorsebatterystaple#c_ppfhfn)

[^1]: [Bonneau paper](https://www.usenix.org/system/files/conference/usenixsecurity14/sec14-paper-bonneau.pdf)

