---
title: "Finally Setting Up HTTPS"
date: 2026-03-17T14:23:48Z
---

Well, it's time.  We shove HTTP into the ditch, and make HTTPS our home.  Or, at least, 301 is the new 200.  (or, 443 is the new 80)

## First, you need snap

Debian 13 does not come with snap pre-installed, but it's in apt, so ...

~~~
apt update
apt install snapd
~~~

I think I should've logged in again at this point.

## Then, you install certbot

~~~
snap install --classic certbot
ln -s /snap/bin/certbot /usr/local/bin/certbot
~~~


## Let's set it up

Well, it seems like it really expects nginx to be installed at the system level, and not inside a Docker container.  So let's do that.

~~~
# certbot --nginx
...
Congratulations! You have successfully enabled HTTPS on https://arh68.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
~~~

### A short trip through hell

So, at this point, I think I was good to go.  I was going to pass system-level nginx through to Docker-level nginx ... `proxy_pass http://127.0.0.1:8000;` and all that jazz.

But I was getting crazy errors in Safari & Firefox.  Links from the `https://` site were going to `http://` urls, even though they were relative URLs (!?) and all the headers looked okay.  The hovered URL looked okay!  Other relative links worked.  Nothing really made sense.  Certain links (only certain ones!) for relative URLs were going to `http://127.0.0.1` which ... I don't know what to say.  Nothing made sense.

I think my issue was port-related.  If it's listening on :80 but you're accessing it through :8000 or whatever, all of the 301s will be wrong.  Something like that.  Ouch.  I'd rather just keep everything :80 and not have mapping issues.

So instead, let's just wire in the system files into the Docker container.

### Debian-side certbot + Docker-side nginx

Certbot seems to touch 2 directories:

1. __/etc/nginx__ to modify the site configuration
2. __/etc/letsencrypt__ to store its own stuff

I have certbot running outside of Docker with its automatic cron job to refresh, and it should update its folder.  I don't want to run certbot in Docker, and I'm not sure why most guides I read prefer it that way.  I want the cron job !  Maybe the dockerized version *does* have that but the guides seem to imply not (after all, wouldn't it need a full init process?  I might dig into that later).

I have no intent on modifying the nginx config.

So I'll try passing *both* through to the nginx container, read-only, and hopefully I'll get 1 nginx instance that serves HTTPS and automatically sees updates to the certbot files.  I'll have to check the logs in a couple weeks.

Docker Compose now looks like

~~~yaml
services:
  blog:
    image: 127.0.0.1:5000/blog68:production
    restart: always
    ports:
    - "80:80"
    - "443:443"
    volumes:
    - /etc/nginx:/etc/nginx:ro
    - /etc/letsencrypt:/etc/letsencrypt:ro
~~~

So, just to be clear, when I set up Let's Encrypt, the Docker Compose was *down* and the system-level nginx was *up* serving an empty directory.  Now Docker is *up* and the system-level nginx has been stopped.  *Disabled*, in fact.  Ideally, certbot will not need to restart nginx any more past this point.  Their documentation seems to indicate that.  I think it can "just" touch its own files and work.  We'll see what the logs say in 60 days, I guess.

## Last steps

Don't be an idiot, and forget to open the firewall.  Haha, wow.

~~~
ufw allow 'Nginx Full'
~~~

... or forget your VPS has its own layer of firewall rules.

It's actually not bad, once you remember all of the steps you forgot to do.

