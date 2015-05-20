---
layout: post
title: Skew-binary random access lists
date: 2015-05-19 19:26
---

This is another post on another data structure, the skew-binary random access list.  This is my favorite data structure; I just think it's ingenious.  While standard binary random access lists (which I just call bforests) take O(log N) time for nearly every operation, a subtle change to the underlying numerical representation can lower the bound for `cons`, `head`, and `tail` to O(1) time.  Not bad.

Like a lot of programmers/math nerds, I've known about binary numbers for a long time.  Modern computers are built on this most simple number system.  They more than make up for their simplicity in their speed, like a 2-stroke engine running at a billion RPMs.  [Skew-binary numbers][wiki-sb], on the other hand, are a bit *strange*.  I had not heard of them before reading Chris Okasaki's [paper][pfral].  Why you would want to build a data structure on them was beyond me.  Soon, I began to appreciate them.

## A slight detour through a bforest

There's something about adding `1` to a binary number that's just a drag.  Try doing long addition on 63, and you end up carrying every bit.

~~~
 111111 (carries)

  111111
+      1
--------
 1000000
~~~

This hits you right in the face when you try to `cons` something onto the front of a bforest.  What seems like *adding just a grain of sand* ends up taking forever  (where forever = O(log N) time).  Adding a `1` on the low side 'bubbles' up all the way to the top.

As it turns out, there is another way.  There are lots of ways, I suppose, but the one I'm showing you today is *very* good.

As it turns out, you can fix this 'bubbling' up.  Instead of bubbling a `1` up to the next highest zero, which could be log N away, what if we just push the bubble up exactly one digit?  Wouldn't that be great?

## Forever blowing bubbles

If we just push the `1` up just one digit adding 1 to 63, we'd actually get a `2`.  This seems kind of inevitable.

~~~
  111111
+      1
--------
  111112
~~~

You might think we're just counting in ternary at this point, where any digit can be `0`, `1`, or `2`.  But not so!  The trick here is that apart from leading zeroes, *only the lowest digit* can be a `2`.  That's all there is to it.

So when we add a `1` again, we can't put it to the right of the `2`, and we can't put a bunch of `2`s in a row: we have to somehow combine it.  Wouldn't it be nice if the `2` just bubbled up exactly once?

~~~
  111112
+      1
--------
  111120
~~~

Would this work on standard binary numbers, where the *i*th digit represents *2<sup>i</sup>*?  Let's find out.  We know `111111` is equal to 63.  Is `111112` equal to 64?  32 + 16 + 8 + 4 + 2 + 2(1) = 64.  That seems to check out.  Is `111120` equal to 65?  32 + 16 + 8 + 4 + 2(2) = 64 again.  That's too bad.

So there's no way to make this work with standard binary weights of 2<sup>i</sup>.  Let's think about this, though: when would 2(something big) + 1 = 1(the next biggest thing)?

~~~
2X[n] + 1 = X[n+1]
~~~

I know `X[0]` has got to start at 1, so let's see where this goes inductively.

~~~
X[0] = 1
X[1] = 2X[0] + 1 = 2(1) + 1 = 3
X[2] = 2X[1] + 1 = 2(3) + 1 = 7
X[3] = 2X[2] + 1 = 2(7) + 1 = 15
~~~

Curious!  A pattern emerges.  I love it when things work out.

~~~
X[n] = 2^(n+1) - 1
~~~

## -1 to the rescue

Instead of using weights of 2<sup>i</sup>, we should be able to use weights of 2<sup>i+1</sup> *- 1*.  Let's see how we things work out trying to add `1` to a long chain of `111111`s.

- `111111` is 1 + 3 + 7 + 15 + 31 + 63 = 120.  The change of weights drastically changes the value.  This is okay.
- `111112` is 2(1) + 3 + 7 + 15 + 31 + 63 = 121.
- `111120` is 2(3) + 7 + 15 + 31 + 63 = 122.  Incredible!  We added one, and the `2` bubbled up exactly once.
- `111200` is 2(7) + 15 + 31 + 63 = 123.
- `112000` is 2(15) + 31 + 63 = 124.  I like where this is going.
- `120000` is 2(31) + 63 = 125.
- `200000` is 2(63) = 126.
- `1000000`, finally, is 127.

Hopefully you can see how the bubbling indeed takes O(log N) time, but it's spread out across log N additions, each addition only pushing the low `2` up by one place value.  This means if we build trees of size 2<sup>i+1</sup>-1 instead of trees of size 2<sup>i</sup>, we can implement the 'subtraction' and 'addition' in O(1) time.  This means `cons` and `tail` can be O(1) operations.

## Trees of a different size

One large question remains: *how do you make a tree of 2<sup>k-1</sup> elements*?  In the bforests, we used complete binary trees that each had 2<sup>k</sup> leaves.

You might remember we only stored values in the leaves, though.  What happens if we store values everywhere?

~~~
       .        
      / \       
(   .     .   ) = 4 elements  (2^k)
   / \   / \    
  +   + +   +   

       +        
      / \       
(   +     +   ) = 7 elements (2^(k+1) - 1)
   / \   / \    
  +   + +   +   
~~~

Perfect!  But how do we order things?  It was simple back when we stored the 4 elements in order along the bottom of the tree.

~~~
       .        
      / \       
(   .     .   )
   / \   / \    
  0   1 2   3   
~~~

Where do these new branch values come in?  I can think of at least a couple different orderings.

~~~
       3        
      / \       
(   1     5   )
   / \   / \    
  0   2 4   6   

       0        
      / \       
(   1     2   )
   / \   / \    
  3   4 5   6
~~~

Intuitively, when we combine `2` arbitrarily large trees with a single `1`, there's pretty much only one easy way to stick them together.

~~~
101...02000000 + 1 = ?

    .         .     
   / \       / \    
  /   \     /   \   + 1 = ?
 /     \   /     \  
/__big__\ /__big__\ 

      ___1___
     /       \
    .         .     
   / \       / \    
  /   \     /   \   
 /     \   /     \  
/__big__\ /__big__\ 
~~~

That ought to do it.  The `2` trees may be big, but it's the `1` that binds them.  What kind of ordering would this give us?  It seems like smallest elements will be at or near the top.  Let's try building up a tree from nothing to see exactly how the ordering comes out.

## Build me up

I'm going to build an sforest with exactly 7 elements.  I'll add the elements 0..6 in reverse so the 6 is at the 'end' and the 0 is at the 'beginning'.  Suppose I've got an sforest with one element, `6`.

~~~
( 6 )
~~~

I'll cons a `5` onto the front, and get a new sforest.

~~~
( 5 6 )
~~~

I'll add a third element, `4`.

~~~
    4
(  / \  )
  5   6
~~~

Along comes a `3`.

~~~
       4
( 3   / \  )
     5   6
~~~

I'll add the `2`, then a `1`.

~~~
    1     4  
(  / \   / \  )
  2   3 5   6
~~~

And now I'll finish up with a `0`.

~~~
      _0_
     /   \
(   1     4   )
   / \   / \  
  2   3 5   6
~~~

Curious!  If that's the ordering you were expecting, congratulations.  It's not what I'd call *obvious*.

## Finders keepers

If `tail` is just `cons` in reverse, I've covered `head`, `cons` & `tail`.  But what about `index`, `update`, and `map`?  I need a random access *list*, not a stack.

The reorganization of the trees means we have to search a bit differently.  We still have to scan our sparse list of trees until our index points inside some tree, but once we've got the right tree, how do we find a given index?

If you look at the ordering, it's not like it's random.  A tree of size 7 has 0 at the root, 1..3 on the left, 4..6 on the right.  A tree of size 127 will have 0 at the root, 1..63 on the left, 64..126 on the right.  A tree of size 2<sup>i+1</sup>-1 will have 0 at the root, 1..2<sup>i</sup>-1 on the left, and 2<sup>i</sup>..2<sup>i+1</sup>-2 on the right.

If you remember the bforests, each bit in the number basically boiled down to a left/right decision.  The same thing applies here: all we have to do is make the right decision at each level, subtract from the index accordingly, and not worry about what happens later.  Elements 2<sup>i</sup> and larger are on the right.  Everything smaller than 2<sup>i</sup> is on the left.  All I have to do to find an index in a tree of size 2<sup>i+1</sup>-1 is check if index >= 2<sup>i</sup>.  If it is, I go right and subtract 2<sup>i</sup> from index.  Otherwise I go left and subtract 1.  (If index is 0, I'm done.)

### A silly example

Sometimes I like to work through uncomfortably large examples.  So let's see if we could find index 300 in 0..2046.  I'm not going to draw a tree that big (I'd need a bigger laptop), but I'll break it down.  Index = 300, Tree size = 2047.

1.  300 in 0..2046: 300 < 1024: go left (-1)
1.  299 in 0..1022: 299 < 512: go left (-1)
1.  298 in 0..510: 298 >= 256: go right (-256)
1.  42 in 0..254: 42 < 128: go left (-1)
1.  41 in 0..126: 41 < 64: go left (-1)
1.  40 in 0..62: 40 >= 32: go right (-32)
1.  8 in 0..30: 8 < 16: go left (-1)
1.  7 in 0..14: 7 < 8: go left (-1)
1.  6 in 0..6: 6 >= 4: go right (-4)
1.  2 in 0..2: 2 >= 2: go right (-2)
1.  0 in 0..0: we are done!

We followed the rules and our index got to 0.  That's how you find index 300 in O(log 2047) time out of 2047 elements: go left left right left left right left left right right.

## Mapping & Iterating

To iterate over each element in order, we start at the root and go down to the left.  Once we hit the bottom, then we start to backtrack and go right.  In this sense, we can 'walk' the tree by *keeping our left hand on the wall*, so to speak.  Imagine tracing your left finger around the outside of the contours of the tree: that's exactly how you map over it.

Instead of the standard binary tree map, where we map the left tree, the root value, then the right tree, our root value actually comes first.  The entire left subtree is smaller than the right, so we map left then right.

~~~javascript
map(fn, tree.left)       fn(tree.value)
fn(tree.value)      -->  map(fn, tree.left)
map(fn, tree.right)      map(fn, tree.right)
~~~

## Finis

This is quite a long post.  Hopefully I didn't botch any examples; it's a learning process.  I'll finish up the SForest code soon, push it to GitHub, and break it down in another post.  Until then, happy hacking.

[wiki-sb]: https://en.wikipedia.org/wiki/Skew_binary_number_system
[pfral]: http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.55.5156
