---
title: "CSS Masonry (Soon)"
date: 2026-03-20T10:43:49Z
---

CSS Masonry isn't here yet.  Except for Safari Technology Preview!

So in a way, it *is* here on my laptop.  And I have to say, it looks *pretty* good.  If the order of items is important, though, it depends on how many columns you've got:

1. 1 column: well, it's not really going to do anything

2. 2 columns: *perfect*, just about, as long as items are approximately the same height.  You might have to scroll back up if there are really long items (relative to the others), but usually you know the next item is on the other side.

3. 3 columns: you kind of need numbering to keep track of what's next

4. 4+ columns: ordering seems hopeless, but the effect is still visually nice

When items start at approximately the same y-coordinate, it can be very hard to distinguish which comes first, though, so you're probably best numbering each item if you really want to keep sense of that.  Ups and down, I guess.  It definitely works best for semi-ordered or unordered lists of small items, but I like how it works for text.  The old `columns: 2` *does* work, of course, but you've got to like manually paginate to get any breaks.  In any case, the more options for typesetting, the better!

## Fallback to grid

What's nice is you can still fall back to the normal grid layout if the browser ignores `display: grid-lanes;`

~~~css
...
    display: grid;
    display: grid-lanes;
    grid-template-columns: ...;
~~~

which seems acceptable.

## So when?

Well, the [MDN page](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout/Masonry_layout#browser_compatibility) links to bug trackers for each browser, and there is definitely ongoing bugfixing.  Been going for years now.  It certainly doesn't seem abandoned, though.  Your guess is at least as good as mine ...
