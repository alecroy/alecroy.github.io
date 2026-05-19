---
title: "CSS Counters"
date: 2026-03-21T15:34:24Z
---

You know how you can make an ordered list in HTML and get, well, numbers?  Like

1.
1.
1. ...

It's not the *only* way to do it, though.

You don't even need to create any elements: you can declare, through CSS, something like virtual elements using `::before` and `::after`.  You can declare them to have content with `{ content: ... }`.

What makes it really interesting is that the content can have *counters*.

## Counters

Counters are numbers that start at 0.  They can be incremented by specific elements & used by the same (or others).

The counters can be reset by other elements, so if you reset the counter in parent elements & use them in children, it's like they're scoped.

So if I have some structure like

~~~html
<div class="parent">
	<p>...</p>
	<p>...</p>
	...
~~~

We can prefix each paragraph with a number, just like an ordered list

~~~css
.parent {
    counter-reset: number;
}
p::before {
    content: counter(number);
}
p {
    counter-increment: number;
}
~~~

How crazy is that ?

Like I knew we *could do it*, but in JavaScript.  This is __declarative__ !  No scripting engine needed, no fixing up stuff when the page loads.

One caveat: it seems like they only update when you *add* nodes, not when you remove them.  So maybe stick to like React-type interfaces if you want to avoid that.  Mixing the declarative & imperative seems to only half-work.

Still, pretty impressive!  I have been rather ... chuffed with what I've learned about CSS recently.
