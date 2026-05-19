---
title: "Personal Reader Mode"
date: 2026-03-29T23:15:34Z
draft: true
---

Sometimes, you just don't like the way an article looks.  You could turn on reader mode, but that's a bit too far.  You just want something in between.

You open up the Web Inspector, and start fixing styles.

## Something like

~~~css
body {
	margin: 1rem auto;
	max-width: 40rem;
}

p {
	font: 400 1.0rem/1.0 'Hiragino Mincho ProN';
	text-align: justify;
	hyphens: auto;
}

pre, code {
	font: 400 0.8rem/1.0 Menlo;
}
~~~

Sometimes you want to just run some JavaScript, though ...

~~~javascript
s = new CSSStyleSheet()
s.replaceSync(`
body {
	margin: 1rem auto;
	max-width: 40rem;
}

p {
	font: 400 1.0rem/1.0 'Hiragino Mincho ProN';
	text-align: justify;
	hyphens: auto;
}

pre, code {
	font: 400 0.8rem/1.0 Menlo;
}
`)
document.adoptedStyleSheets.push(s)
~~~
