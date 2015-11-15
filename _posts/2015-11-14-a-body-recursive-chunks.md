---
layout: post
title: "A body-recursive chunks"
date: 2015-11-14 20:02
---

Real quick: I just wanted to add some closure to the [various implementations]({% post_url 2015-11-09-simple-profiling-in-erlang %}) of a ```chunks``` function I wrote in Erlang a few days ago.  I had pretty much settled on a tail-recursive solution that seemed to work reasonably well enough.  But that was before I became enlightened regarding [body recursion]({% post_url 2015-11-12-recursion-in-erlang %}).  So I rewrote it (now 20% fewer lines, and with more flair ```;)``` ).

Out with the old:

~~~Erlang
chunks(L, N) -> lists:reverse(chunks(L, N, [])).
chunks(L, N, Out) when length(L) > N ->
    {Chunk, Rest} = lists:split(N, L),
    chunks(Rest, N, [Chunk | Out]);
chunks(L, _N, Out) -> [L | Out].
~~~

And in with the new shiny shiny:

~~~Erlang
chunks(List, ChunkSize) when length(List) < ChunkSize -> [List];
chunks(List, ChunkSize) ->
    {Chunk, Rest} = lists:split(ChunkSize, List),
    [Chunk | chunks(Rest, ChunkSize)].
~~~

I'd put this version in my [```misc.erl``` utility belt module](http://erlang.org/pipermail/erlang-questions/2011-May/058768.html).  I guess I might want two versions, eventually: one that includes the partial chunk at the end & another that doesn't; I'll cross that bridge when I get there.

~~~Erlang
1> misc:chunks(lists:seq(1, 10), 3).
[[1,2,3],[4,5,6],[7,8,9],"\n"]
~~~