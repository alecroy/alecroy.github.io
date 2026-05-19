---
title: "YAML minus JSON"
date: 2026-03-28T23:01:05Z
draft: true
---

Following up on loading configuration files, let's talk about JSON & YAML.

JSON's pretty convenient in the JavaScript land, of course.  YAML's just one `require` away, though, and I think lots of other languages are at that point as well.

YAML 1.2 has several different schemas.  *One* of them is 'json'.  It supports *some* of what YAML offers, but not all.

Which begs the question: what lies outside the JSON schema?  And what lies in between the two?

Probably a basic question for a lot of people, but ... I guess I don't know the corner cases *that* well, so

I should say, [eemeli](https://eemeli.org/yaml/#document-options) has a really good npm package & documentation, and furthermore he links straight to [the YAML spec](https://yaml.org/spec/1.2-old/spec.html#id2803231) which is arguably what really matters.

## So

As far as I can tell, the interface for `JSON.parse(...)` and `YAML.parse(..., { schema: 'json' })` match success-for-success and failure-for-failure.  The difference, then, is between the JSON schema and the Core schema; what the Core schema adds ...

| type | JSON schema | Core schema adds |
| ---- | ----------- | ------------------ |
| nulls | null | Null NULL ~ |
| booleans | true false | True TRUE False FALSE |
| integers | 12345 -67890 | +12345, octals (0o1237), hexadecimals (0x123F) |
| floats | 1.234E±56 -.7890 | +1.234 ±.inf ±.Inf ±.INF .nan .NaN .NAN |

It is notable that `+123` and `+123.456` are not valid JSON.  I think we've all known to some degree that `NaN` and `Inf` aren't JSON, but if you're reading/writing YAML that won't be a problem.

Also note that 'No' is no longer a boolean as of YAML 1.2 which came out kind of a while ago.

So overall I'd say YAML has a pretty decent standing in the configuration field.  I don't know if there's one perfect config language to rule them all and in the darkness bind them &c &c, but it's pretty good.
