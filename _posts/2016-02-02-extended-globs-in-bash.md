---
layout: post
title: "Extended globs in bash"
date: 2016-02-02 20:10
---

I learned about extended globbing today.  I wanted to free up some disk space on my tiny macbook, so I ran a scan with [Grand Perspective](https://en.wikipedia.org/wiki/GrandPerspective).  I saw an unusually large file in the git repo for Fira that I cloned (really I just wanted the `.otf` files, but cloning the whole thing seemed easier).  Somewhere in the `Fira` directory lurked a ~300 MB file.  I just wanted to verify that's where it was, nothing fancy.

So the goal is: show regular entries (`README.md`) and dotted/hidden entries (`.git`) altogether.

`ls` shows hidden files, but doesn't show you anything about *size*, of course:

```text
9281 ~/Dropbox/Fonts/Fira (master u=)
ls -la
total 88
drwxr-xr-x@  17 dude  staff   578 Oct 25 12:39 .
drwxr-xr-x@ 108 dude  staff  3672 Dec  6 20:10 ..
-rw-r--r--@   1 dude  staff  6148 Oct 25 12:40 .DS_Store
drwxr-xr-x@  12 dude  staff   408 Feb  2 19:05 .git
-rw-r--r--@   1 dude  staff  4515 May 31  2015 LICENSE
-rw-r--r--@   1 dude  staff   169 May 31  2015 README.md
-rw-r--r--@   1 dude  staff    65 May 31  2015 bower.json
drwxr-xr-x@  37 dude  staff  1258 May 31  2015 eot
-rw-r--r--@   1 dude  staff  7379 May 31  2015 fira.css
-rw-r--r--@   1 dude  staff  8128 May 31  2015 index.html
drwxr-xr-x@  37 dude  staff  1258 May 31  2015 otf
-rw-r--r--@   1 dude  staff   618 May 31  2015 package.json
drwxr-xr-x@   4 dude  staff   136 Oct 25 12:39 source
drwxr-xr-x@  38 dude  staff  1292 May 31  2015 technical reports
drwxr-xr-x@  37 dude  staff  1258 May 31  2015 ttf
drwxr-xr-x@  37 dude  staff  1258 May 31  2015 woff
drwxr-xr-x@  37 dude  staff  1258 May 31  2015 woff2
```

It says `.git` is `408 bytes`, but Grand Perspective determined that was a lie. (of course, `ls` is just showing the size of the directory entry, not the total size)

My go-to `du -hs ./*` didn't pull up `.`-directories, which I knew included the main offender:

```text
9282 ~/Dropbox/Fonts/Fira (master u=)
du -hs ./*
8.0K    ./LICENSE
4.0K    ./README.md
4.0K    ./bower.json
5.2M    ./eot
8.0K    ./fira.css
8.0K    ./index.html
 11M    ./otf
4.0K    ./package.json
101M    ./source
 54M    ./technical reports
 13M    ./ttf
6.1M    ./woff
4.4M    ./woff2
```

I couldn't find a `-a` option for `du`, so I needed something else.  Of course, `./*` itself doesn't match hidden directories.  I can use the glob `./.*`, but then I *only* get hidden directories.  I wanted them all in one listing:

```text
9284 ~/Dropbox/Fonts/Fira (master u=)
du -hs ./.*
513M    ./.
654M    ./..
8.0K    ./.DS_Store
319M    ./.git
```

## extglob & dotglob

There are a couple ways to get to where I wanted:

1. use the `dotglob` shell option, which will change the globbing convention to *include* matching an initial dot

2. use the `extglob` shell option to give you better globs

I prefer the second option.  Here's the first, for completeness:

```text
9280 ~/Dropbox/Fonts/Fira (master u=)
shopt -s dotglob

9281 ~/Dropbox/Fonts/Fira (master u=)
du -hs ./*
8.0K    ./.DS_Store
319M    ./.git
8.0K    ./LICENSE
4.0K    ./README.md
4.0K    ./bower.json
5.2M    ./eot
8.0K    ./fira.css
8.0K    ./index.html
 11M    ./otf
4.0K    ./package.json
101M    ./source
 54M    ./technical reports
 13M    ./ttf
6.1M    ./woff
4.4M    ./woff2
```

This is nice since it excludes the `.` and `..` entries automatically, but I'd rather include those by default and glob them out by hand.  Here's all there is to know about extended globs:

1. they can describe regular languages, which makes them regular expressions
2. they don't look like regular expressions (the PCRE kind)
3. their syntax is pretty simple (from `man bash`):

```text
?(pattern-list)
     Matches zero or one occurrence of the given patterns
*(pattern-list)
     Matches zero or more occurrences of the given patterns
+(pattern-list)
     Matches one or more occurrences of the given patterns
@(pattern-list)
     Matches one of the given patterns
!(pattern-list)
     Matches anything except one of the given patterns

A pattern-list is a list of one or more patterns separated by a |.
```

So for example, `*(foo)*` will match `foo.md`, `foofoo.md`, and `bar.md`, but `+(foo)*` will not match `bar.md`.

## The du I was looking for

So armed with `shopt -s extglob`, here we go:

```text
9283 ~/Dropbox/Fonts/Fira (master u=)
du -hs ./?(.)*
513M    ./.
654M    ./..
8.0K    ./.DS_Store
319M    ./.git
8.0K    ./LICENSE
4.0K    ./README.md
4.0K    ./bower.json
5.2M    ./eot
8.0K    ./fira.css
8.0K    ./index.html
 11M    ./otf
4.0K    ./package.json
101M    ./source
 54M    ./technical reports
 13M    ./ttf
6.1M    ./woff
4.4M    ./woff2
```

I can exclude the `.` and `..` entries with the extended glob `./!(.|..)`:

```text
9289 ~/Dropbox/Fonts/Fira (master u=)
du -hs ./!(.|..)
8.0K    ./.DS_Store
319M    ./.git
8.0K    ./LICENSE
4.0K    ./README.md
4.0K    ./bower.json
5.2M    ./eot
8.0K    ./fira.css
8.0K    ./index.html
 11M    ./otf
4.0K    ./package.json
101M    ./source
 54M    ./technical reports
 13M    ./ttf
6.1M    ./woff
4.4M    ./woff2
```

Finally, a proper `du`.  I don't know how I went so long without this.  It makes `ls -a` look kind of weird and unnecessary.  I can get fancier, now, and start sorting / limiting:

```text
9290 ~/Dropbox/Fonts/Fira (master u=)
du -ms ./!(.|..) | sort -n | tail -5
11  ./otf
14  ./ttf
54  ./technical reports
101 ./source
319 ./.git
```

## Finis

You only get extended globs with bash 3+.  Right now, OS X comes with 3.2.57 and you can install 4.3.42 from homebrew, so no looking back on my macbook.  Don't forget to turn on `extglob`, since it's not enabled by default.  (you can add it to your `.bash_profile` to use it in all new terminals)

```text
9280 ~
echo 'shopt -s extglob' >> ~/.bash_profile
```

```text
9281 ~
shopt extglob # in a new terminal
extglob         on
```

Happy hacking!  Glob on.