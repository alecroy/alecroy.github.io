---
layout: post
title: "Filenames suck"
date: 2015-12-01 18:25
---

I really like bash scripts.  Who doesn't, right?  But as the saying goes, *a man's got to know his limitations*.  One thing I've never liked is how you have to handle filenames on the shell.

Have you ever had a file named ``` ```?  Or maybe ```-l```?  How would you ```ls``` that file if you didn't know ahead of time to expect it?  Would all your shell scripts handle them cleanly?  Do you even know which utilities would fail?  I don't.

Spaces and dashes in filenames are a pain in the ass, more or less, but that's just the *tip* of the iceberg.  I made a note to myself a few weeks ago: __newlines in filenames?__.  I wondered if I would ever come across such a .. monstrosity.  I know you shouldn't display filenames that could contain control characters, but it's something you almost never run into; off the top of my head I just wasn't sure.

So I went to Wikipedia and looked up [Ext4][ext4].  It's a pretty reasonable filesystem.  Let me just quote the wiki here:

~~~text
Allowed characters in filenames:
All bytes except NUL ('\0') and '/' and the special file names "." and ".."
~~~

Mother of god.  Why is there such a wide disparity between what filesystems *support* and what normal people *actually use*?  How did we get to here?

There's a long dwheeler essay on [Fixing POSIX Filenames][dw-posix] that sums up the situation really well.  He's written more about it than I've ever read.  I think I have some O'Reilly books shorter than that post.  He points to a [comment by epa on LWN][epa] that notes filenames might not even contain valid Unicode bytestrings.  You can't even guarantee a filename will be valid UTF-8, UTF-32, or *any* reasonable subset of all bytestrings (except what the filesystem disallows).

For what it's worth, here's what a couple other reasonable filesystems allow (according to [Wikipedia][w]):

| Filesystem | Filename restriction |
| ---------- | -------------------- |
| [HFS+][h] | 255 UTF-16 units, no restrictions (may contain ```\0```!?). |
| [NTFS][n] | 255 UTF-16 units, no ```/``` or ```\0```.  On Win32, none of <code>\ : * ? " < > &#124; "</code> . |
| [ZFS][z] | 255 ASCII characters, no ```\0```. |

I really wonder how you'd handle a filename with a ```\0``` in it.  Not even trusty ```find ... -print0 | xargs -0 ...``` would get that right.  Even writing in C wouldn't help (as long as you rely on C-strings).  More on language choice:  there's a good quote by Wittgenstein: __The limits of my language mean the limits of my world__.  When your language is bash, you tend to stay away from nasty filenames with spaces and newlines.  When your language is C, you tend to stay away from strings containing multiple ```\0``` bytes.  Don't take the word *limit* too seriously; conventions (like null-terminated strings in C, Unicode strings in Python, et cetera) are usually as important as limits.

If there are tools out there that handle *all* filenames, they're probably written in languages that make strings-as-arbitrary-bytestrings easier than Bash/C have made them.  And again, they'd have to *not display filenames*.  Time to start reading (& writing) in hex, I guess.  It's the only way to be safe.  ```;)```

As wide-open as their alphabets are, so to speak, it *is* a bit surprising to see such a low limit on length.  255 might seem huge, but it's ```TEXTFI~1.TXT``` all over again when you consider URLs can contain [thousands of characters][url] (not that it's a good idea to use so many).

Anyhow, filenames suck.  Happy hacking.


[ext4]: https://en.wikipedia.org/wiki/Ext4
[dw-posix]: http://www.dwheeler.com/essays/fixing-unix-linux-filenames.html
[epa]: http://lwn.net/Articles/325304/
[w]: https://en.wikipedia.org/wiki/Comparison_of_file_systems#Limits
[h]: https://en.wikipedia.org/wiki/HFS_Plus
[n]: https://en.wikipedia.org/wiki/NTFS
[z]: https://en.wikipedia.org/wiki/ZFS
[url]: http://boutell.com/newfaq/misc/urllength.html
