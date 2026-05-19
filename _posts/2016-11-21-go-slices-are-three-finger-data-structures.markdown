---
layout: post
title:  "Go slices are three finger data structures"
date:   2016-11-21 16:00:00 -0400 EDT
categories: Software
---

What Go devs call slices are what I think of as a "three-finger data structure".

Coming from a C background, I know pointers well, and they're basically a one-finger data structure.  They point somewhere; that's it.  They track one dimension.  (let's ignore `sizeof`, though I guess that might pass for a dimension)

Usually, the next step up the ladder of complexity is the two-finger/two-dimension data structure.  Now I don't mean matrices; everything here is stall an arrayish listish data type.  I'm talking about pointer + length.  This contrast, between one- and two-finger data structures, can be clearly seen with the null-terminated-C-strings vs finite-length-Pascal-strings.

Many programmers are comfortable not just with one-finger data structures, but also two-finger variants.  What I don't think comes familiar to a lot of us, though, are the three-finger ones.  Not saying we can't learn, but we don't see them all the time.
      
What would the finger point to?  Does the second finger change?  What possibilities, unseen before, can arise?

## 1 finger: C strings
- your thumb marks the first element

## 2 fingers: Pascal strings
- your thumb marks the first element
- your index finger marks the last element (Length!)

## 3 fingers: Go slices
- your thumb marks the first element
- your index finger marks the last element you care about (Length!)
- your pinky marks the last element that's known to exist (Capacity!)

This means you can actually expand a slice in Go by moving your index finger towards your pinky.  In code, this would look like:

~~~go
s = s[:cap(s)]
~~~

If it's not obvious, you may only grow slices rightwards, never leftwards.

By default, subslices will keep the capacity of the (larger) slice.

~~~go
s := [100]int{}
x := s[:5]
// x's thumb = s's thumb
// x's length is 5
// x's capacity is 100
~~~

To limit the capacity to the length (to prevent expanding the slice later),

~~~go
s := [100]int{}
x := s[:5:5] // [thumb : index : pinky]
// x's thumb = s's thumb (defaults to 0)
// x's length is 5 (index - thumb)
// x's capacity is 5 (pinky - thumb)
~~~
