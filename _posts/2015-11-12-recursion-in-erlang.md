---
layout: post
title: "Body & tail recursion in Erlang"
date: 2015-11-12 20:12
---

Old habits die hard.  Some things can suprise you.

I was writing a couple small libraries in Erlang over Veterans Day.  Random access lists, my favorite.  I'd already written them in JavaScript, but why not give it a whirl.  This time I called them [binary_forest](https://github.com/alecroy/binary_forest) & [skew_forest](https://github.com/alecroy/skew_forest).

As a data structure built largely with *lists*, I wasn't surprised to use some recursion.  Due to the way *I* learned functional programming, I tend to use tail recursion.  I mean, *who wouldn't*, right?  What this entails is usually breaking one function up into two: a non-recursive wrapper that gets exposed/exported & a helper that uses tail recursion.

This looks something like:

~~~OCaml
let my_double l =
  let rec helper l acc = match l with
  | [] -> List.rev acc
  | h::t -> helper t ((2*h)::acc) in
  helper l [];;

my_double [1; 2; 3];;

- : int list = [2; 4; 6]
~~~

That's OCaml, not Erlang, but the idea remains the same.  The actual implementation of ```my_double``` is just a single call to ```helper```.  ```helper``` usually takes an additional argument, often an *accumulator* that it builds up as it recurses.  The most efficient way to build up a list is to add to the *beginning*, not the end, so the first element processed actually ends up at the end of the accumulated list; one call to ```reverse``` and we're set.  This is pretty common in Lisp/Scheme, too; not just OCaml.  This seemed like the natural way to write Erlang; *functional programming* is *functional programming*, right?

## Body recursion, enter stage left

I came across the [Eight Myths of Erlang Performance][myths].  The third myth stood out to me: __Myth: Tail-Recursive Functions are Much Faster Than Recursive Functions__.  Why would that be?  How could that be?

I read [a StackOverflow answer by Hynek -Pichi- Vychodil][so], and [a blog post by Fred Hebert][ferd].  That pretty much sealed the deal.  The mailing list Fred [links to][mailing_list] is a good read.  Body recursion is well-optimized & avoids the final list reversal (which takes time and memory).

I tested it myself, read what others have written, and I've got to agree: *body recursion is fast*.  Body recursion does have the distinct benefit of being easier to understand (no more helper functions, no more reversing), so I rewrote my code.

Here's what it looked like before (this from binary_forest):

~~~Erlang
update_trees(N, Value, Trees) -> update_trees(N, Value, Trees, []).
update_trees(N, Value, [Tree = #tree{size=Size} | Trees], Out) when N > Size ->
    update_trees(N - Size, Value, Trees, [Tree | Out]);
update_trees(N, Value, [Tree | Trees], Out) ->
    lists:reverse([update_tree(N, Value, Tree) | Out]) ++ Trees.
~~~

And after:

~~~Erlang
update_trees(N, Value, [Tree = #tree{size=Size} | Trees]) when N =< Size ->
    [update_tree(N, Value, Tree) | Trees];
update_trees(N, Value, [Tree = #tree{size=Size} | Trees]) ->
    [Tree | update_trees(N - Size, Value, Trees)].
~~~

The second version just seems much more *obvious*.  Here are the commits: one for [binary_forest][binary_commit], one for [skew_forest][skew_commit].  I think I'll stick to body recursion if I can help it.


[cs3110]: http://www.cs.cornell.edu/courses/cs3110/2015fa/l/03-var/rec.html
[myths]: http://www.erlang.org/doc/efficiency_guide/myths.html
[so]: http://stackoverflow.com/questions/2475270/how-to-read-the-contents-of-a-file-in-erlang/2475613#2475613
[ferd]: http://ferd.ca/erlang-s-tail-recursion-is-not-a-silver-bullet.html
[mailing_list]: http://erlang.2086793.n4.nabble.com/Efficiency-of-a-list-construction-tp3538122p3538379.html
[binary_commit]: https://github.com/alecroy/binary_forest/commit/6c4e0eea9d662d17a347de38763e91299dc2ef9d
[skew_commit]: https://github.com/alecroy/skew_forest/commit/733fea8697adbf9e29cde58e0a212f6cc12fb0e9
