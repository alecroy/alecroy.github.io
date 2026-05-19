---
title: "Composing Docker Compose"
date: 2026-03-25T23:09:46Z
---

Let's take a look at [merges](https://docs.docker.com/reference/compose-file/merge/), [extends](https://docs.docker.com/reference/compose-file/services/#extends), & [includes](https://docs.docker.com/reference/compose-file/include/).  I have heard of them, but have not had a good reason to reach for them yet.  About all I know is that you can specify multiple compose files, and they get combined in some way.

I'm wondering:

1. how services are (or aren't) namespaced: if 2 *compositions*, let's say, have a service named `db`, is that a problem ?

1. can services "know" about each other without being explicitly referenced, or is there some way to explicitly reference them ?

1. can I instantiate the same composition (K) times with different names?  (like if I want 4 identical twin type processes)

1. can you pass things down into sub-compositions?  like if I want them all to pull images with a certain tag like `2026-03-25` or `staging` or whatever

Let's "delve" into it ...

## Merges

Notably,

> By default, Compose reads two files, a compose.yaml and an optional compose.override.yaml file.

I had no idea!  That's wild.  I mean, it's not much when you can load multiple files, but still.

Merges are order-dependent.  Subsequent files override earlier ones.  All paths are relative to the *first* file.  Overrides can be "just" fragments, not complete compose files.

Singular/scalar options are overwritten.  Plural/vector options are concatenated.  

## Extends

I think I like extends best.  They're not as "global" as the other 2 techniques.  If you want to override something, but you don't know *where* you want the override to happen, I think you want the extends.

Their example, as they explain, *does not* create a `webapp` service but simply reuses the properties of that service as defined in the `common-services.yaml`

~~~yaml
services:
  web:
    extends:
      file: common-services.yml
      service: webapp
~~~

You can explicitly create a `webapp` service to mimic, of course

~~~yaml
services:
...
  webapp:
    extends:
      file: common-services.yml
      service: webapp
~~~

Notably,  you can extend services defined in the same file (like to share environment).  You could also use YAML anchors for small cases, I guess?



## Includes

Notably,

> Compose displays a warning if resource names conflict and doesn't try to merge them.

> include applies recursively

> Any volumes, networks, or other resources pulled in from the included Compose file can be used by the current Compose application for cross-service references.

> Compose also supports the use of interpolated variables with include.

~~~yaml
include:
  -${INCLUDE_PATH:?FOO}/compose.yaml
~~~

There is a short syntax:

~~~yaml
include:
  - ../commons/compose.yaml
~~~

and a long syntax:

~~~yaml
include:
   - path: ../commons/compose.yaml
     project_directory: ..
     env_file: ../another/.env
~~~

with an explicit project directory & `.env` for interpolated variables.  It seems the short syntax assumes the compose file is in the project directory, and the environment file is in that project directory.



## docker compose config

Another thing I did not actually know is that you can more or less dry-run the composition to see what exactly gets included/extended/whatever with `docker compose -f docker-compose.yaml  config`.  It'll even expand shorthands/defaults.


The only question I had that doesn't look answered is looping.  I guess I should've expected coding-in-YAML to not have a for loop.  It's not too bad to type out all 4 services that extend the same base service, then apply different names/ports.  Or maybe it's an anti-pattern and you shouldn't do it in the first place!  Either way, I think I've satisfied my curiosities.

There is a lot more to compose than I realized.  You can still glue together your own YAML, but it's pretty sophisticated in its, like, YAML combinators (if you will).
