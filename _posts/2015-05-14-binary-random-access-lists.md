---
layout: post
date: 2015-05-14 21:23
title: "BForest, a binary random access list"
---

This is a post on a data structure, the binary random access list.  Not the best data structure in the world, but close to one of my favorites.  I read about them a couple years ago in a [paper][pfral] by Chris Okasaki.  I recently picked up his book [Purely Functional Data Structures][pfds] at [Powell's Books][powells], so I've been wanting to write them up in JavaScript.  There's more than one kind of random access list mentioned in the paper; this one is the simplest.

I wrote an implementation of this data structure in JavaScript and posted it on GitHub.  I called the library [BForest][gh-bf] for a couple reasons:

1.  It's a collection of trees, binary trees, of various sizes.  A *forest*, so to speak.
1.  binary-random-access-list is a pain to say.
1.  "bral" isn't much better.
1.  There's no BForest already in npm to get confused with.
1.  It's derived from [binary numerical representations][wiki-binary].  When I write a [skew-binary][wiki-skew] version, I'll probably call it SForest.
1.  The only other prevalent reference to 'forest' in CS involves [spanning trees][wiki-spanning-tree], a completely separate topic.
1.  I can't help calling them forests, and I don't know how to say 'forest' in anything but English.

## The Idea

Hopefully you're already familiar with binary trees and linked lists.  Binary trees, of which you usually only have one, can contain many elements: 1 tree of size N.  Here's 1 tree of size 6.

~~~
    +
   / \
  +   +
 / \   \
+   +   +
~~~

Linked lists, on the other hand, have N elements each of size 1.  If you consider a single element as a tree of size 1, then here's 6 trees of size 1.

~~~
( +  +  +  +  +  + )
~~~

Random access lists are a happy medium between the two: a linked list of logarithmic size containing binary trees of logarithmic size.  Here's 6 elements in a binary random access list.

~~~
    .        .        
   / \      / \       
( +   +   .     .   ) 
         / \   / \    
        +   + +   +   
~~~

Note that these trees are a bit different.  Values (`+`) are only stored in the leaves, never up in the branches (`.`).  Also, the trees only come in sizes of *powers of 2*.  That's what makes them *complete* binary trees, as they are completely full.

If you think about it, the number `6` is written `110` in binary.  One `4`, one `2`, and no `1`.  The size of the trees corresponds exactly to the digits in the binary numerical representation.  This is why they are called *binary* random access lists (not because they use binary trees).

Any number can be written in binary, so any number of elements can be stored in a binary random access list.

### Build me up

Adding an element to the list is just like adding `1` to a binary number: the `1` element carries other `1`s up until it finds an empty slot.  Unlike binary trees, where you can insert elements into the middle, we'll just be inserting to the *front* of the list (*cons*ing).  Let's add an element to the list of 6.

~~~
    .        .                  .        .
   / \      / \                / \      / \ 
( +   +   .     .   ) -> ( +  +   +   .     .   )
         / \   / \                   / \   / \
        +   + +   +                 +   + +   +
~~~

That was easy.  Adding `1` to `110` would naturally lead to `111`.  Let's add another, bubbling the 'carries' up step by step.

~~~
          .        .
         / \      / \ 
( +  +  +   +   .     .   )
               / \   / \
              +   + +   +
~~~

~~~
    .      .        .
   / \    / \      / \ 
( +   +  +   +   .     .   )
                / \   / \
               +   + +   +
~~~

~~~
       .           .
      / \         / \
(   .     .     .     .   )
   / \   / \   / \   / \
  +   + +   + +   + +   +
~~~

~~~
         ____.____  
        /         \
       .           .
      / \         / \
(   .     .     .     .   )
   / \   / \   / \   / \
  +   + +   + +   + +   +
~~~

Adding 1 to 7 (`111`) makes 8 (`1000`).

### Break me down

Just like consing, we'll only remove elements from the *front* of the list.  This is pretty easy: in the best case, there's a single tree of size 1 just waiting to be plucked.

~~~bash
( +  .. )  ->  ( .. )
~~~

In the worst case, we take have to subtract from a tree of size N.  This is like subtracting 1 from a power of 2: `1000..0` becomes `0111..1`.  Much like how adding 1 to 7 reduced 3 trees down to 1, breaking apart a large tree generates lots of smaller trees.  Just imagine going from the list of `8` back up towards the list of `7`.  I'll label the 3 trees we're saving as `A`, `B`, and `C`.  `X` marks the element we're removing.

~~~
         ____.____ 
        /         \
       .           C                    B        C
      / \         / \                  / \      / \
(   .     B     .     .   )  ->  ( A  +   +   .     .   )
   / \   / \   / \   / \                     / \   / \
  X   A +   + +   + +   +                   +   + +   +
~~~

### Indexing

Random access lists, given the name, should allow random access.  To look up a given index, we have to first figure out which tree it will be in.  A tree of size `S` will store indices `0 .. S-1`.  There are a logarithmic number of trees (as many as a binary number is wide), so just *finding* the right tree in a BForest of N elements will take O(log N) time.  The biggest tree we find could be as big as the entire forest (a forest of a single tree, like `N=8`), so finding the element in the correct tree will also take up to O(log N) time.

If that part doesn't make sense, let me explain it a little more intuitively.  When you write down a number in any base, it has a certain *width*.  100,000 is a *six digit figure*, as the saying goes.  It's six digits wide in base 10.  When you write 7 in binary as `111`, it's 3 digits wide.  This width is the *logarithm*.  A BForest of 7 elements will have 3 trees like we saw above, so it might take as long to find the last tree as the number 7 is wide.

Taking this a bit further, notice that the `C` tree above has 3 levels: 1 with 4 elements (the ones holding the values), 1 with 2 elements (branches), and 1 with 1 element (the `C`).  Since it'll take 3 steps just to get to the bottom, it takes as long to walk down the largest tree as the number 7 is wide.

Adding the 2 operations (finding the tree, walking the tree), an index operation takes twice logarithmic time.  Due to math not worth talking about here, 2 \* O(log N) = O(log N); the index operation takes O(log N) time.

### Updating

Updating is basically the same as indexing, except we want to replace one element with another.  In some languages, you'd just set `element.value = newValue;`, but this is a *purely functional* data structure.  Once you create a BForest, it will not change through operations like `head`, `tail`, `cons`, `prepend`, `index`, or `update`.  These operations will return a new forest (or a single element) and leave the original forest intact.

This complicates the operation, as we have to create a new tree just to have a new leaf that points to a new value.  Most of the tree carries over, but the entire chain of parents has to be replaced.

~~~
    .        A        
   / \      / \       
( 1   2   .     B   ) 
         / \   / \    
        3   4 X   6   
~~~

To replace element `X` with `Y`, we have to replace everything that could point down to `X`.  None of the other values or branches will need to change.  `B` will always point to `X`, and so will `A`, so first we'll create a new parent `Q` to replace `B`.

~~~
  B        Q  
 / \  ->  / \ 
X   6    Y   6
~~~

Likewise for `A`, we replace `A` with `P` and copy its structure, pointing to `Q` instead of `B`.

~~~
       A                  P       
      / \                / \      
(   .     B   ) -> (   .     Q   )
   / \   / \          / \   / \   
  3   4 X   6        3   4 Y   6  
~~~

There's very little 'work' we have to do in each step, but there are as many steps as the tree is tall.  Plus it takes time to find the right tree.  So like index, the update operation takes O(log N) time.

## The Code

I'll step through the code I wrote (as of this morning).  It's only [137 sloc][bforest-js], though it could probably be a lot shorter.  I wrote it in vanilla JavaScript without libraries (though lodash might've been nice).

### Laying out the data

A BForest is essentially an array.  In that array are trees.  Trees are hashes with 3 or 4 fields: `size`, `left`, `right`, and optionally a `value`.  The `left` and `right` fields should be trees, themselves (or null).  A tree of size 1 looks like `{size: 1, value: 123, left: null, right; null}`.  Branches look like `{size: somePowerOfTwo, left: leafOrBranch, right: leafOrBranch}`.

### The Constructor

All bforests are constructed similarly.  `new BForest()` will create an empty forest.  For convenience, `new BForest(array)` will create a new forest and preload the array into it.  It is equivalent to `new BForest().prepend(array)`.

~~~javascript
var BForest = function(array) {
  this.trees = [];
  if (array !== undefined) {
    this.trees = this.prepend(array).trees;
  }
};
~~~

It's useful to know when a forest is empty.  BForests are empty when their array is empty.

~~~javascript
BForest.prototype.isEmpty = function() {
  return (this.trees.length === 0);
};
~~~

### Head & Tail

The head of a BForest will be the left-most element on the left-most tree.  It doesn't make much sense to call head on an empty forest, so I return `null` in that case.

~~~javascript
BForest.prototype.head = function() {
  if (this.isEmpty()) { return null; }

  var ptr = this.trees[0];
  while (ptr.left !== null) {
    ptr = ptr.left; // Go all the way left
  }
  return ptr.value;
};
~~~

Tail is a bit more complicated, as we have to return a valid BForest that's different from the original.  This is where we 'subtract' 1.  It's always the first tree that we break apart, and it's always the left-most leaf we omit.  The leaves/branches on the right sides of the first tree should be collected and grouped with the 2nd, 3rd, .. trees in the BForest.

I collect trees from the top down, (`C`, `B`, `A` according to the earlier example), so I `.reverse()` them before merging with the 2nd, 3rd, .. trees.

~~~javascript
BForest.prototype.tail = function() {
  if (this.isEmpty()) { return this; }

  var bf = new BForest();
  if (this.trees[0].size > 1) { // Need to break apart first tree
    var ptr = this.trees[0];

    while (ptr !== null && ptr.right !== null) {
      bf.trees.push(ptr.right); // Collect the trees on the right
      ptr = ptr.left; // Recurse down the left side of the tree
    }

    bf.trees.reverse(); // Last tree pushed on is the smallest/'first'
  }

  bf.trees = bf.trees.concat(this.trees.slice(1)); // Keep old trees
  return bf;
};
~~~

### Cons

Cons is the opposite of tail.  This is where we 'add 1'.  We get away easy if the forest doesn't have any trees of size 1.  In that case, we just add a tree of size 1 to the forest.

I create a new tree and bubble it up (adding other trees to it) until I find no other trees of the same size (a `0` in the number).  Once I find that `0`, I can merge the new tree in with the later ones.

~~~javascript
BForest.prototype.cons = function(element) {
  var bf = new BForest();
  var newTree = { size: 1, value: element, left: null, right: null};

  for (var i = 0; i < this.trees.length; i++) {
    if (this.trees[i].size > newTree.size) {
      bf.trees = [newTree].concat(this.trees.slice(i));
      return bf;
    }
    newTree = {size: 2 * newTree.size, left: newTree, right: this.trees[i]};
  }

  bf.trees = [newTree];
  return bf;
};
~~~

If I bubble up all the trees in the forest, I end up with just one tree, the new one.  That's those last 2 lines.

### Index & Update

Indexing works in two steps: finding the right tree, then finding the element within that tree.  The important thing to remember is that since a BForest is just an array of trees, any slice of that array also represents a valid BForest.  For example, if `(1 2 4)` represents a BForest of 7 elements, `(2 4)` has 6 elements, and `(4)` has 4.  The only difference is that these suffixes are smaller in size, so indices need to be adjusted down as you progress.  For example, indices 0-6 are valid on `(1 2 4)`, but only 0-3 are valid on `(4)`.  So accessing index 6 of `(1 2 4)` is the same as accessing index 5 of `(2 4)` and index 3 of `(4)`.

~~~javascript
BForest.prototype.index = function(index) {
  if (this.isEmpty() || !Number.isInteger(index) || index < 0) { return null; }

  for (var i = 0; i < this.trees.length; i++) {
    if (index < this.trees[i].size) { // It's in this tree
      var ptr = this.trees[i];
      while (ptr.left !== null) {
        if (index < ptr.left.size) { // 0 <= index < 2^n
          ptr = ptr.left; // [0..2^n] -> [0..2^(n-1)]
        } else {
          index -= ptr.left.size; // [0..2^n] -> [2^(n-1)..2^n] - 2^(n-1)
          ptr = ptr.right;
        }
      }
      return ptr.value; // We must be at the leaf [i] if ptr.left is null
    } else { // It's in a later tree, subtract from index # skipped
      index -= this.trees[i].size;
    }
  }
  return null; // We looked at all the trees and didn't find [i]
};
~~~

Update works very similarly.  Scan the forest for the tree that needs to go.  Take it out, replace it, then merge it back in with the others.  I wrote a recursive helper function to update the tree for me, using the stack to start with the parents, expand the call stack down to the leaves, then bubble leaf updates back up towards the top.

~~~javascript
function updateTree(tree, index, value) {
  if (tree.size === 1) {
    return {size: 1, left: null, right: null, value: value};
  }

  var newTree = {size: tree.size, left: tree.left, right: tree.right};
  if (index < tree.left.size) {
    newTree.left = updateTree(tree.left, index, value);
  } else {
    newTree.right = updateTree(tree.right, index - tree.right.size, value);
  }
  return newTree;
}

BForest.prototype.update = function(index, value) {
  if (this.isEmpty()) { return null; }

  for (var i = 0; i < this.trees.length; i++) {
    if (index >= this.trees[i].size) { // It's in a later tree
      index -= this.trees[i].size; // Subtract from index # skipped
      continue;
    }

    var bf = new BForest();
    bf.trees = this.trees.slice(0, i); // Collect all trees before
    bf.trees.push(updateTree(this.trees[i], index, value));
    bf.trees = bf.trees.concat(this.trees.slice(i + 1)); // And after
    return bf;
  }
  return this; // Updated nothing! Return the original tree
};
~~~

As you can see on this last line, out-of-bounds updates do nothing.

### Iter & Map

What good is an array-like data structure if you can't iterate over it?  Not much.  So I wrote two functions for doing this: 

- iter(someFn) runs someFn on every element in order and returns nothing
- map(transformFn) runs transformFn on every element in order and returns a new BForest with the transformed elements

So if you just want to print the thing as-is, iter is the way to go.  If you want to transform all the elements, map is the way to go.  These two functions are the only way to modify a BForest destructively.  The element passed to the functions (someFn, transformFn) can be modified (not replaced, though; that's what update is for), modifying the original BForest (and all other BForests that point to that element).  Needless to say, you probably shouldn't destructively update the elements unless you really intend to.

~~~javascript
function iterTree(fn, tree) {
  if (tree.left === null) { // Leaf, call the function
    fn(tree.value);
  } else {
    iterTree(fn, tree.left); // Left, then right
    iterTree(fn, tree.right);
  }
}

BForest.prototype.iter = function(fn) {
  for (var i = 0; i < this.trees.length; i++) {
    iterTree(fn, this.trees[i]);
  }
};
~~~

The main difference between iter and map is that map constructs a new BForest from the leaves up and iter does not.

~~~javascript
function mapTree(fn, tree) {
  if (tree.size === 1) {
    return {size: 1, value: fn(tree.value), left: null, right: null};
  } else {
    var newLeft = mapTree(fn, tree.left);
    var newRight = mapTree(fn, tree.right);
    return {size: tree.size, left: newLeft, right: newRight};
  }
}

BForest.prototype.map = function(fn) {
  var bf = new BForest();

  for (var i = 0; i < this.trees.length; i++) {
    bf.trees.push(mapTree(fn, this.trees[i]));
  }

  return bf;
};
~~~

### Pretty printing

I sure love me some properly formatted output.  I added a toString() method that prints a BForest just like an array.  You'll have to see the tests (test/bforest_test.js) to see more.

~~~javascript
BForest.prototype.toString = function() {
  var strings = [];
  this.iter(function(elt) { strings.push(elt.toString()); });
  return '[' + strings.join(', ') + ']';
};
~~~

## Finis

That's it!  Thanks for reading this far down.  I'll soon get started on the skew-binary version, because they should be a bit faster.  Happy hacking.

[pfral]: http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.55.5156
[pfds]: http://www.powells.com/biblio/61-9780521663502-1
[powells]: http://www.powells.com
[wiki-binary]: https://en.wikipedia.org/wiki/Binary_number
[wiki-skew]: https://en.wikipedia.org/wiki/Skew_binary_number_system
[wiki-spanning-tree]: https://en.wikipedia.org/wiki/Spanning_tree#Spanning_forests
[bforest-js]: https://github.com/alecroy/bforest/blob/37b18cacc947b6a77329703321c8a42a3ddf5aaf/bforest.js
[gh-bf]: https://github.com/alecroy/bforest
