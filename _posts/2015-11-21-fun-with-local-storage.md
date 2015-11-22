---
layout: post
title: "Fun with localStorage"
date: 2015-11-21 18:00
---

I've been reading through the source of [SC4][sc4], a self-contained crypto web application.  It's like PGP in a .html file.  You use one HTML page (`sc4z.html`) to generate another (randomly named, like `SC4_9EzCzXFM2UARR9.html`) and you're pretty much all set.

As part of the standalone page, SC4 uses `localStorage` to store keys.  This is fine when you're serving the content from a normal domain (`https://origin.tld/path/`), but what I didn't know is that different browsers handle this differently when it's served from a `file://` URI.

This is mentioned in the SC4 security audit, but it's news to me.  Firefox will store a separate `localStorage` database for each directory; in other words, only files located within the same directory can access the same `localStorage` database.  Chrome works differently, though: it stores *all* data for `localStorage` for `file:` URIs in a *single* database.  What this means is that *any* file served from a local filesystem can read the `localStorage` database.

I did some further digging and found that `data:` URIs are handled differently from the `file:` URIs, too.


[sc4]: https://github.com/Spark-Innovations/SC4
