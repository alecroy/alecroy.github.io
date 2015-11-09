---
layout: post
title: "Simple profiling in Erlang"
date: 2015-11-09 18:00
---

Over the weekend, I was looking around the ```lists``` module for a function that could group a list into pairs or triplets.  Either a ```pairs```, ```triplets```, ```group_by```, or ```chunks``` function, basically.  I didn't find one, so I thought I'd give it a go myself.  There are [many][py-1] [ways][py-2] [to][py-3] [do][py-4] [this][py-5] in Python, some faster than others, so I wanted to know how to do it *fast* in Erlang.

There's more than one way to [profile/benchmark][profiling] code in Erlang.  There's [eprof][eprof], which records a table of functions called & percent of time spent in each function.  There's [fprof][fprof], which records hierarchical profiling information (what called what, how much time was spent in each).  There's [tc][tc], which just records wall clock time spent evaluating a function (like ```$ time ./script```, but in the Erlang shell).

I'm not striving for accuracy here, just curiosity.  I'm not running this on multiple machines or multiple architectures.  I'm not going to close my e-mail & web browsers.  I'm not even going to launch new Erlang processes for each evaluation.  *Your mileage may vary.*

## A first try

Here's my initial attempt at a ```chunks``` function.  I thought I'd make it as general as possible.  Looking through the [```lists```][lists] module, ```nthtail``` and ```sublist``` seemed to fit.

~~~Erlang
chunks(L, N) -> chunks(L, N, []).
chunks(L, N, Out) when length(L) > N ->
    chunks(lists:nthtail(N, L), N, [lists:sublist(L, N) | Out]);
chunks(L, _N, Out) -> lists:reverse([L | Out]).
~~~

Running this through ```fprof```, grouping a list of 10,000 into chunks of 250, I get about ~19,000 function calls and ~500 ms of runtime.

~~~
1> Args = [List, ChunkSize] = [lists:seq(1, 10000), 250].
[[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,
  23,24,25,26,27,28|...],
 250]
2> fprof:apply(chunks, chunks, Args), fprof:profile(), fprof:analyse([no_details]). 
Reading trace data...
.............................
End of trace!
Processing data...
Creating output...
%% Analysis results:
{  analysis_options,
 [{callers, true},
  {sort, acc},
  {totals, false},
  {details, false}]}.

%                                               CNT       ACC       OWN        
[{ totals,                                     19642,  557.029,  555.407}].  %%%


Done!
ok
~~~

## A second try

I thought I'd give list comprehensions a try, since I can reuse ```sublist``` to pull out an inner list (instead of starting at the first element).

~~~Erlang
chunks2(L, N) -> [lists:sublist(L, I, N) || I <- lists:seq(1, length(L), N)].
~~~

This has the benefit of being a *one-liner*, which I really like.  Let's see the (condensed) ```fprof``` output.

~~~
1> Args = [List, ChunkSize] = [lists:seq(1, 10000), 250].
2> fprof:apply(chunks, chunks2, Args), fprof:profile(), fprof:analyse([no_details]).

%                                               CNT       ACC       OWN        
[{ totals,                                     205289, 3808.218, 3804.196}].  %%%
~~~

That's a lot slower, regardless of timing variance.  Ten times the number of function calls.  I could omit the ```no_details``` to see the entire call hierarchy, but since it's not in my code (it's in ```lists:sublist/3```), there's not much I can do about it.

## Third time's the charm




[py-1]: https://stackoverflow.com/questions/312443/how-do-you-split-a-list-into-evenly-sized-chunks-in-python
[py-2]: https://stackoverflow.com/questions/434287/what-is-the-most-pythonic-way-to-iterate-over-a-list-in-chunks
[py-3]: https://stackoverflow.com/questions/1624883/alternative-way-to-split-a-list-into-groups-of-n
[py-4]: https://stackoverflow.com/questions/4119070/how-to-divide-a-list-into-n-equal-parts-python
[py-5]: https://stackoverflow.com/questions/9671224/split-a-python-list-into-other-sublists-i-e-smaller-lists?lq=1

[profiling]: http://www.erlang.org/doc/efficiency_guide/profiling.html
[eprof]: http://www.erlang.org/doc/man/eprof.html
[fprof]: http://www.erlang.org/doc/man/fprof.html
[tc]: http://www.erlang.org/doc/man/timer.html#tc-3

[lists]: http://www.erlang.org/doc/man/lists.html