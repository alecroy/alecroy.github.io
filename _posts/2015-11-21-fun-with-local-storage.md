---
layout: post
title: "Fun with localStorage"
date: 2015-11-22 00:26
---

I've been reading through the source of [SC4][sc4], a self-contained crypto web application.  It's like PGP in a .html file.  You use one HTML page (`sc4z.html`) to generate another (randomly named, like `SC4_9EzCzXFM2UARR9.html`) and you're pretty much all set.

As part of the standalone page, SC4 uses `localStorage` to store keys.  This is fine when you're serving the content from a normal domain (`https://origin.tld/path/`), but what I didn't know is that different browsers handle this differently when it's served from a `file://` URI.

This is mentioned in the SC4 security audit, but it's news to me.  Firefox will store a separate `localStorage` database for each directory; in other words, only files located within the same directory can access the same `localStorage` database (including subdirectories, apparently).  There is a good page on the [Mozilla Developer Wiki][mozwiki] detailing the Same-Origin Policy for `file:` URIs.  Chrome works differently, though: it stores *all* data for `localStorage` for `file:` URIs in a *single* database.  What this means is that *any* file served from a local filesystem can read the `localStorage` database.

There is a [lengthy discussion on Mozilla's Bugzilla][mozbugs] about how `localStorage` should work with `file:` URIs.  It's not a recent bug; the original, more-Chrome-like implementation [was discussed][mozbugs09] back in 2009.  If you try accessing `localStorage` from a separate directory, you won't get the SC4 keys; you won't get anything.  In *Chrome*, though, you __will__.  This isn't a huge security concern, but it's a little bit surprising.  HTTP-served resources are restricted by domain (according to the same origin policy), but `file:` URIs are the wild west, so to speak.  Go ahead and `console.log` the `localStorage` from any local file in Chrome: you'll get anything & everything other files store.

I did some further digging and found that `data:` URIs are handled even differently from the `file:` URIs.  I was wondering if the SC4 pages could be served entirely from `data:` URIs (humongous links, more or less), but it appears to be not so.  Long story short: they don't support `localStorage`.  I don't have a good narrative *why* this is so (there are discussions on bug trackers), but it's not outrageous.

This is pretty uniform across all browsers.  There's a [Chromium bug][chromium] marking this `working as intended`, so I don't expect any changes soon.  Here are some error messages that come up when you try to reference `localStorage` from a `data:`-served resource:

#### Chrome

```
SecurityError: Failed to read the 'localStorage' property from 'Window': Storage is disabled inside 'data:' URLs.
```

#### Firefox

```
[Exception... "Component is not available" nsresult: "0x80040111 (NS_ERROR_NOT_AVAILABLE)" location: "JS frame :: data:text/html;base64,PGh0bWw+CjxoZWFkPgo8bWV0YSBjaGFyc2V0PSJ1dGYtOCI+CjwvaGVhZD4KPGJvZHk+CjxzY3JpcHQgdHlwZT0idGV4dC9qYXZhc2NyaXB0Ij4KdHJ5IHsgZG9jdW1lbnQud3JpdGVsbihKU09OLnN0cmluZ2lmeShsb2NhbFN0b3JhZ2UpKSB9CmNhdGNoIChlKSB7IGRvY3VtZW50LndyaXRlbG4oZSkgfQo8L3NjcmlwdD4KPC9ib2R5Pgo8L2h0bWw+Cg== :: :: line 7" data: no]
```

#### Safari

```
Error: SecurityError: DOM Exception 18
```

Here's the HTML I used; it's visible in the Firefox error message.

```html
<html>
<head>
<meta charset="utf-8">
</head>
<body>
<script type="text/javascript">
try { document.writeln(JSON.stringify(localStorage)) }
catch (e) { document.writeln(e) }
</script>
</body>
</html>
```

Encoding this in base64 yields:

```text
PGh0bWw+CjxoZWFkPgo8bWV0YSBjaGFyc2V0PSJ1dGYtOCI+CjwvaGVhZD4KPGJvZHk+CjxzY3JpcHQgdHlwZT0idGV4dC9qYXZhc2NyaXB0Ij4KdHJ5IHsgZG9jdW1lbnQud3JpdGVsbihKU09OLnN0cmluZ2lmeShsb2NhbFN0b3JhZ2UpKSB9CmNhdGNoIChlKSB7IGRvY3VtZW50LndyaXRlbG4oZSkgfQo8L3NjcmlwdD4KPC9ib2R5Pgo8L2h0bWw+Cg==
```

So the `data:` URI is `data:text/html;base64,PGh0...snipped...Cg==`.

I'm *really* not sure how Internet Explorer handles all this, but I'm not exactly inclined to find out.  We'll see.  I like SC4, `localStorage` or not.

[sc4]: https://github.com/Spark-Innovations/SC4
[mozbugs09]: https://bugs.webkit.org/show_bug.cgi?id=20701
[mozbugs]: https://bugzilla.mozilla.org/show_bug.cgi?id=507361
[mozwiki]: https://developer.mozilla.org/en-US/docs/Same-origin_policy_for_file%3A_URIs
[chromium]: https://code.google.com/p/chromium/issues/detail?id=304828