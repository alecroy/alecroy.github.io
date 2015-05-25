---
layout: post
title: "SForest, another random access list"
date: 2015-05-25 15:57
---
In [a recent post][skewpost], I discussed the mechanics underlying skew-binary random access lists.  In this post, I'll walk through some code I've written to bring the ideas to life with JavaScript.

## The library

[SForest][sforest] is a purely functional data structure similar to [BForest][bforest].  Both use linked lists of binary trees to provide O(log N) indexing and updating.  SForests improve the runtime of adding & removing elements from the front of the list down to O(1) time.

I want to quickly walk through the JavaScript I wrote to make this data structure work.  I wrote the code to be obvious and clear, so it's not quite production-ready yet.  I don't use any libraries or frameworks, just vanilla JavaScript.

### It begins with a Constructor

Like the BForest constructor, this one takes an optional array argument that sets the initial contents of the SForest.  Passing `[]` or nothing at all (`undefined`) creates an empty SForest.

~~~javascript
function SForest(array) {
  this.trees = [];

  if (array !== undefined) {
    this.trees = this.prepend(array).trees;
  }
}
~~~

This relies on the `.prepend(..)` method which I'll get to later.  I also use a `.isEmpty()` helper method just like in BForest.

~~~javascript
SForest.prototype.isEmpty = function() {
  return (this.trees.length === 0);
};
~~~

### Head, Tail, Cons

Additions and deletions are only possible from the front of the forest.  Getting the head is very simple as it should be in the root of the first tree.  Note that unlike the BForest's `.head(..)`, this is clearly O(1).

~~~javascript
SForest.prototype.head = function() {
  if (this.isEmpty()) {
    return null;
  }

  return this.trees[0].value;
};
~~~

Tail isn't too bad, especially if there's a tree of just 1 element out front.  In that case, I create a new SForest with all the trees *but* that one.  If there's no 1-tree out front, I break apart the first tree into three parts: its root, its left branch, and its right branch.  I keep the branches (and the other trees) and omit the root.

~~~javascript
SForest.prototype.tail = function() {
  if (this.isEmpty()) {
    return this;
  }

  var sf = new SForest();
  if (this.trees[0].size === 1) {
    sf.trees = this.trees.slice(1);
  } else {
    var firstTwoTrees = [this.trees[0].left, this.trees[0].right];
    sf.trees = firstTwoTrees.concat(this.trees.slice(1));
  }
  return sf;
};
~~~

Unlike the BForest version that involved walking trees, this looks like it is going to run O(1) as long as `Array.slice(1)` runs in constant time.

Cons works much the opposite of tail: ideally, I can add a 1-tree out in front if there's not already a pair of same-size trees.  If there is a pair, I combine the pair as left & right branches into a bigger tree, using the new element as root.

~~~javascript
SForest.prototype.cons = function(element) {
  var sf = new SForest();
  var tree;

  if (this.trees.length < 2 || // Don't even have two trees, add a 1
    this.trees[0].size < this.trees[1].size) { // Two trees, different sizes
    tree = { size: 1, value: element, left: null, right: null };
    sf.trees = [tree].concat(this.trees);
  } else { // A leading 2, a pair of trees of the same size
    tree = {
      size: this.trees[0].size * 2 + 1,
      value: element,
      left: this.trees[0],
      right: this.trees[1],
    };
    sf.trees = [tree].concat(this.trees.slice(2));
  }

  return sf;
};
~~~

I use an `Array.slice(2)` here but that shouldn't keep it from running in O(1) time.

### Prepend

Once `cons` exists, `prepend` comes pretty naturally.  The only non-obvious part is loading the elements *in reverse*.  You want the last element to go in first: to make an SForest from the array `[1, 2, 3]`, you want to build [], 3 cons [], 2 cons 3 cons [], then finally 1 cons 2 cons 3 cons [].

~~~javascript
SForest.prototype.prepend = function(array) {
  if (array === undefined) {
    return this;
  }

  var sf = this;
  for (var i = array.length - 1; i >= 0; i--) {
    sf = sf.cons(array[i]);
  }
  return sf;
};
~~~

Since the `cons` operation is O(1), and an N-element array generates N `cons` operations, this should take O(N) time.

### Indexing & Updating

Indexing is where we do the complicated left/right decisions, decreasing our index with each move.  Going left always decreases the index by 1, and going right basically *halves* the index.  Unlike BForests, you can stop early up in the branches (above the leaves), as there are lots of values up there.  Once the index has been decreased to 0, we stop looking.

There are log N trees to search through, then a maximum depth of log N to find the value.  Decrementing the index on each level should take constant time.  In general, doing arithmetic on the tree sizes could take log N time (for truly gargantuan sizes), but for all sizes under 2<sup>64</sup> it should only take a few instructions on most hardware.

~~~javascript
SForest.prototype.index = function(index) {
  if (this.isEmpty() || !Number.isInteger(index) || index < 0) {
    return null;
  }

  for (var i = 0; i < this.trees.length; i++) {
    if (index < this.trees[i].size) { // It's in this tree
      var ptr = this.trees[i];
      while (index > 0) {
        if (index >= (1 + ptr.size) / 2) { // Go right, -2^i
          index -= (1 + ptr.size) / 2;
          ptr = ptr.right;
        } else { // Go left, -1
          index -= 1;
          ptr = ptr.left;
        }
      } // Index is 0, stop searching
      return ptr.value;
    } else {
      index -= this.trees[i].size;
    }
  }

  return null; // Looked at every tree and never found [i]
};
~~~

The update function is probably the messiest.  I split it up into a recursive helper function like in BForest.  The `.update(..)` method scans for the tree that needs updating, calls out to `updateTree(..)` for a replacement, then splices the replacement tree into a new SForest.

~~~javascript
SForest.prototype.update = function(index, element) {
  if (this.isEmpty() || !Number.isInteger(index) || index < 0) {
    return this;
  }

  for (var i = 0; i < this.trees.length; i++) {
    if (index < this.trees[i].size) { // It's in this tree
      var sf = new SForest();
      sf.trees = this.trees.slice(0, i); // Before
      sf.trees.push(updateTree(this.trees[i], index, element)); // New tree
      sf.trees = sf.trees.concat(this.trees.slice(i + 1)); // After

      return sf;
    } else {
      index -= this.trees[i].size;
    }
  }

  return this; // Looked at every tree and never found/updated [i]
};
~~~

The `updateTree(..)` function uses the stack to descend the tree, building up new branches that point down to the new leaf and all the other, old leaves.  I use the same indexing methodology to navigate down to the index to be replaced.

~~~javascript
function updateTree(tree, index, element) {
  var newTree = {
    size: tree.size,
    value: tree.value,
    left: tree.left,
    right: tree.right,
  };

  if (index === 0) {
    newTree.value = element;
  } else {
    var leastOnRight = (1 + newTree.size) / 2;
    if (index >= leastOnRight) {
      newTree.right = updateTree(newTree.right, index - leastOnRight, element);
    } else {
      newTree.left = updateTree(newTree.left, index - 1, element);
    }
  }

  return newTree;
}
~~~

### Mapping & Iterating

Like the recursive helper function `updateTree(..)`, I use helper functions for mapping and iterating: `mapTree(..)` & `iterTree(..)`.

The only difference between `.iter(..)` and `.map(..)` is that map applies its function to build up a new SForest while iter just applies its function to the existing forest.

~~~javascript
SForest.prototype.iter = function(fn) {
  for (var i = 0; i < this.trees.length; i++) {
    iterTree(fn, this.trees[i]);
  }
};

SForest.prototype.map = function(fn) {
  var sf = new SForest();
  for (var i = 0; i < this.trees.length; i++) {
    sf.trees.push(mapTree(fn, this.trees[i]));
  }
  return sf;
};
~~~

The helper functions do most the work.  Since the root of each tree holds the first value, the left branch holds the small indices, and the right branch holds the large indices, I iterate over `tree.value` before `tree.left`, then `tree.right`.

~~~javascript
function iterTree(fn, tree) {
  fn(tree.value); // Trees always have values
  if (tree.left !== null) {
    iterTree(fn, tree.left); // Left, then right
    iterTree(fn, tree.right);
  }
}
~~~

Similarly, I build the trees up in `mapTree(..)` in the same order.

~~~javascript
function mapTree(fn, tree) {
  if (tree.size === 1) {
    return {size: 1, value: fn(tree.value), left: null, right: null};
  }

  var newValue = fn(tree.value);
  var newLeft = mapTree(fn, tree.left);
  var newRight = mapTree(fn, tree.right);
  return {size: tree.size, value: newValue, left: newLeft, right: newRight};
}
~~~

## One last thing

I wanted to verify the example I used in the previous post.  Finding the index #300 in a tree of indices #0..2046 is not something I can do in my head.  I worked it out by hand last time, and it looked right, but I couldn't be sure.  Some things work in theory, but not in practice.  Now I can add some console logs to see it work or fail.  First, the `app.js` code snippet:

~~~javascript
function range(from, to) {
  var arr = [];
  for (var i = to; i >= from; i--) {
    arr.unshift(i);
  }
  return arr;
}

var _2047 = new SForest(range(0, 2046));
console.log(_2047.index(300));
~~~

Here's the logging modification done to `.index(..)`:

~~~javascript
...
      while (index > 0) {
        if (index >= (1 + ptr.size) / 2) { // Go right, -2^i
          index -= (1 + ptr.size) / 2;
          ptr = ptr.right;
          console.log('Go right');
        } else { // Go left, -1
          index -= 1;
          ptr = ptr.left;
          console.log('Go left');
        }
      } // Index is 0, stop searching
...
~~~

And finally, the output:

~~~javascript
Go left
Go left
Go right
Go left
Go left
Go right
Go left
Go left
Go right
Go right
300
~~~

## Finis

That's it for today!  If I can clean up the code to production-quality, I will publish it on npm.  Until then, you can grab it from GitHub.  Happy hacking!

[sforest]: https://github.com/alecroy/sforest
[bforest]: https://github.com/alecroy/bforest
[skewpost]: http://alecroy.me/2015/05/19/skew-binary-random-access-lists
