---
layout: post
title: "MathJax is So Easy to Install"
date: 2019-12-23T10:12:18-05:00
categories: Blogging
params:
  math: true
---

> *(How Easy? __So__ Easy)*

Wow, I had no idea.  It takes like 2 minutes to add MathJax to your site, if you use their CDN.
<!--more-->

It's pretty much what you'd think up if you had to spit out a stupid-simple install:

1. you add the CDN include at the bottom of your `<body>`

```
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
```

2. you write `$$math_{jax} * stuff = anywhere$$` and it renders as:

	$$math_{jax} * stuff = anywhere$$

If you run Hugo, just consult their docs.  It works super well.

## Inline MathJax

If you try to write it inline (`like $$x+2$$ so`) it shows up like $$x+2$$ so, which really isn't bad, and is probably what I'd do with pen & paper anyway.  Markdown syntax highlighting does get confused by a single asterisk, though, so there might be a better way.

### Dumb mistakes

I just now realize `$$ .. $$` is intended to signify BLOCK MODE and `$ .. $` may be intended to signify INLINE MODE.[^0]  You learn, like Alanis says.

[The docs](https://docs.mathjax.org/en/latest/basic/mathematics.html) are much more useful:

> *The default math delimiters are `$$...$$` and `\[...\]` for displayed mathematics, and `\(...\)` for in-line mathematics. Note in particular that the `$...$` in-line delimiters are not used by default. That is because dollar signs appear too often in non-mathematical settings, which could cause some text to be treated as mathematics unexpectedly.*

I guess that makes sense why I couldn't get `$..$` working, either.

I must report from the future that I actually kind of like the `$...$` inline style, especially since I "turn on" MathJax per-post up in the YAML/TOML front matter.  It's not bad to escape a dollar sign as `\\$`, anyhow.

## Downsides

Right now, there's a weird `TypeError: cannot convert null to object` from `tex-mml-chtml.js:1:32937` which I am __not__ going to look into.  It kind of ruins the fast-as-crap responses I'm used to, but I could maybe probably:

  -  work to get back to wrapping the MathJax sources into my blog output, so they're hosted on this domain

  -  work to only put the source includes on pages that have actual math on them, if possible

  -  work to put that into a JavaScript snippet, and calculate the sha256/base64 for an updated Content-Security-Policy

Or I could just *bet on the cache* and use the CDN.  Maybe my new take will be: __browsers should come with MathJax__ and pulling it in from a CDN in the meantime is just the polyfill we need in this world.

## Alternate includes

Maybe I need to find a different, or more slimmed-down include.

[MathJax says](https://www.mathjax.org/#gettingstarted) I need this:

    https://polyfill.io/v3/polyfill.min.js?features=es6

Do I?  Is that the best `?features=` option?

It says I need:

    https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js

Is that the only thing I can include?  I don't know.  I know there's a bit of lag sometimes, just live-reloading between saves.

### The plot thickens

The plot added corn starch.

No, really, the MathJax Getting Started says:

> *Note: the configuration file tex-mml-chtml.js is a great way to test both TeX and MathML input options at once. You can find [leaner combined configuration packages](http://docs.mathjax.org/en/latest/web/components/combined.html) in our documentation.*

AKA Jackpot, Baby.

Reading on, [MathML](http://docs.mathjax.org/en/latest/basic/mathematics.html#mathml-input) is some weird (not bad! just XML/SGML whatever) input format I most certainly won't be typing any time soon [^1].  So I could maybe subtract that out of my include.

To review, there are 8 configs to choose from:

    tex-chtml      <-- likely a better default for me
    tex-chtml-full <-- tempting..
    tex-svg
    tex-svg-full
    tex-mml-chtml  <-- default..
    tex-mml-svg
    mml-chtml
    mml-svg

I could probably drop down to the `tex-chtml` or down/up to `tex-chtml-full`.  I'll go down, knowing I can go to full if I want.  Great!

I'm tempted to remove the polyfill include, but I don't mind at the moment, unless it annoys me later.  Right now, the `tex-chtml.js` resource is 766.84 KB, 167.39 KB transferred.  I could drop that single file out in my `./static/` directory, and include it in every single post (but not in lists).

I could even just leave the default *off* (so there is no impact at all to pages that don't use it, like __true bypass__), and make blog posts that use math include a `<script ..></>` tag themselves.  I actually did that by accident, trying to demo how to include the CDN `<script>` tags.

If you just write a `<script>` tag in your Markdown, it'll get included in the output[^2].

__MathJax: highly recommended!__

[^0]: https://github.com/jgm/pandoc/issues/2976  it hurts to be stupid, sometimes

[^1]: I'd have to learn it first!

[^2]: assuming you turned on the option to pass raw HTML in Hugo since v0.60 or whatever, though if I were you I'd rewrite all of the raw HTML out of there and put the script in a partial that only conditionally includes it (via a parameter in the front matter), like a more modern Hugo setup would recommend
