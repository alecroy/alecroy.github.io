---
title: "Canvas performance outline"
date: 2026-03-26T23:27:02Z
---

One thing not yet clear to me is how to *best* draw things on a JavaScript `<canvas>` in the 2-D mode.

There are *so* many options and I've never benchmarked any of them.

When I wrote my clock, I initially drew every frame basically the simplest way I could:

1. clear the whole frame

1. draw each part, one by one

I rewrote it to use multiple off-screen canvases, so that in each frame:

1. clear the whole frame (ugh)

1. use `drawImage` to draw the background

1. for each hand of the clock, rotate & draw those images, too

But is it even better for this use case?  I don't know.  CPU usage isn't terrible, though I think when running it at 60 fps it could probably be better.  I'm starting to think that given the very few line segments & arcs, I should probably have kept it the first way.  But off-screen canvas is a neat technique, and I'm glad to have learned it, so ...


## Methods

So what kinds of methods do we have?

1. clearing: clearing the whole frame, or clearing (several) smaller redraw regions, or "un-drawing" each element as the background color

1. drawing: drawing directly, or rotating & drawing pre-rendered off-screen canvases, or some mix of the 2

In my use case, I'm looking at: low number of line segments & arcs (roughly 20–120, depending on how much I reuse), high resolution (I think it looks better at 2000x2000 than 200x200, but I could revise it), few colors (roughly 4), few redraw regions (but awkward to state as rectangles, since they'd need rotation &c).

I do want to test all 4 dimensions, so to speak, to know how the "*best*" would change accordingly.

## Resources

There are a few write-ups about what's good and what's not.  Unfortunately, jsperf seems to no longer exist (GONE?).

1. [web.dev's article](https://web.dev/articles/canvas-performance)

1. [chrome for developer's article](https://developer.chrome.com/blog/canvas2d/)

1. [some framework](https://github.com/Shirajuki/js-game-rendering-benchmark?tab=readme-ov-file) [rendering benchmarks](https://github.com/slaylines/canvas-engines-comparison)

## JavaScript benchmarking (DIY?)

How hard can it be to subtract one timer from another ?  And average the last (N) ?

I know the privacy what-not will possibly round the best timers, but presuming I can turn that off for benchmarking, is there anything else?

It seems like my goal will be to use [Performance.now()](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now) and ensure I'm using [cross-origin isolation](https://developer.mozilla.org/en-US/docs/Web/API/Window/crossOriginIsolated), which surely won't be a problem?

I've got some benchmarking work to do.