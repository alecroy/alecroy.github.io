---
title: "CSS Grid does not disappoint"
date: 2026-03-18T14:23:26Z
---

I cannot complain about CSS Grid.  Even without Layer 3, Layer 2 is really good.  

It's the layout framework we should've had like 20 years ago.  But I'm just glad it's here now.

You get ...

1. an arbitrary number of grid lines, like the yellow snap lines you'd find in Pages.app or some desktop publishing app like InDesign or whatever

1. vertical *and* horizontal lines, naturally (hence the *grid* name)

1. an ability to label/name those lines, plus a default numbering scheme (positive going forward, *and* negative going backward!)

1. a powerful way to define distances between those lines, including both fixed and fractions of what's left (like 2 parts this, 1 part that, ...), plus a few min/max bounding functions

1. a convenient way to place elements at specific places in the grid (from some northwest coordinate across & down to any lines east & south)

1. automatic placement of all remaining elements in 2+ directions (left->right, top->bottom, or ... "dense"-ly)

1. consistent gaps / gutters sizable in both directions (as if the grid lines had width)

Like what more could we want?

It's about as good a layout DSL as I've seen yet.  The exact verbiage can be a little weird, but like anything worth learning you get used to it.  Pretty usable considering *it's all in text* (no visual editor input).

I built my little [π typer game thing](/pi) with about 3 grids.  I'm not much of a designer, so it looks pretty basic, but the alignment and spacing is basically exactly what I wanted.  No more `.row` divs for every row in the grid ... no more resigning to the `<table>` element ...

CSS Grid does not disappoint.
