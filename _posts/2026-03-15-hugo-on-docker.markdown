---
title: "Hugo on Docker"
date: 2026-03-15T14:31:57Z
---

I just want to write down how the blog works these days.  It's a five step process: create, develop, build, test, deploy.  It could probably be set up better, but it's good enough.

## How the blog runs

1. `new.sh` — create a new post

~~~bash
local DATE=$(date -I)

docker run --interactive --rm             \
           -v $(pwd):/site                \
           --workdir /site                \
           ghcr.io/gohugoio/hugo:v0.157.0 \
           new content ./content/posts/${DATE}-"$@".markdown
~~~

1. `dev.sh` — run it locally in develop mode

~~~bash
docker run --interactive --rm             \
           -p 1313:1313                   \
           -v $(pwd):/site                \
           --workdir /site                \
           ghcr.io/gohugoio/hugo:v0.157.0 \
           serve --buildDrafts            \
                 --bind 0.0.0.0
~~~

1. `build.sh` — build the Docker image

~~~bash
local DATE=$(date -I)

docker build -t 127.0.0.1:5000/blog68:${DATE}    \
             --platform linux/amd64,linux/arm64  \
             .

docker tag 127.0.0.1:5000/blog68:${DATE}         \
           127.0.0.1:5000/blog68:production

docker push 127.0.0.1:5000/blog68:production
~~~

The Dockerfile is pretty unambitious, "just" the normal `nginx` image, no Alpine anything.  The size runs ~250 MB, which is fine.  Only the final layer should be changing, anyways.

~~~Dockerfile
from ghcr.io/gohugoio/hugo:v0.157.0 as builder
user 1000
workdir /site
copy --chown=1000:1000 . /site
run hugo build

from nginx as server
copy --from=builder /site/public /usr/share/nginx/html
expose 80/tcp
~~~

1. `test.sh` — run the Docker image

~~~bash
docker run --interactive --rm     \
           -p "127.0.0.1:1313:80" \
           127.0.0.1:5000/blog68:production
~~~

1. `deploy.sh` — deploy the Docker image

~~~bash
ssh the-blog-server                                 \
    docker compose --file /root/docker-compose.yaml \
                   up                               \
                   --pull always                    \
                   --detach
~~~

The neat part about the deployment is that I've got my local Docker registry running on my laptop, so when I RemoteForward the port to the server, the server can pull down the image through the registry that appears as localhost.

~~~yaml
services:
  blog:
    image: 127.0.0.1:5000/blog68:production
    restart: always
    ports:
    - "80:80"
~~~

The RemoteForward bit is in `~/.ssh/config` with the key config.

~~~
Host ...
	...
	IdentityFile ...
	RemoteForward 5000 127.0.0.1:5000
~~~

## Docker on Debian x64 13 (Trixie)

I guess the most annoying thing is that my VPS doesn't come with Docker installed.  [Docker tells you how to install it.](https://docs.docker.com/engine/install/debian/)  Not too bad.
