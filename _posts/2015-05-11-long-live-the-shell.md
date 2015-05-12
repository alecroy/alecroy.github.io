---
layout: post
date:   2015-05-11 18:30
title:  "Long live the shell"
---

There are few things in this world you can count on like `bash`.  It's had its [bad moments][shellshock], but it's the one tool you can almost guarantee is installed *everywhere*.  `bash` is the [Technics SL-1200](https://en.wikipedia.org/wiki/Technics_SL-1200) of shells.  Before you even run your Ruby/Python/Node.js interpreter, you're probably in `bash` already.

I don't think I'll ever *stop* learning new things bash can do.  I'll try to write down in this post some of the things I can remember learning & using.  I won't get into *writing* bash scripts in this post, just the mechanics of being on the command line.

### Typographic Conventions

To avoid confusion below: when I say '`C-a`', I mean 'press `Control` plus `a`'.  When I say '`C-a C-e`', I mean 'press `Control` plus `a`, release everything, then press `Control` plus `e`'.  When I say '`M-b`', I mean 'press `Meta` plus `b`'.

#### More on `Meta`

If you run Linux, `Meta` is probably your Alt key.  If you run OS X, open Terminal > Preferences > Keyboard and check the box next to `Use Option as Meta key`.  `Option` will be your `Meta` key.  If you don't know what `Meta` is, tapping `Esc` should mimic holding down `Meta`.  Note that you don't have to hold down `Esc`: press it once and it will wait for and consume your next keystroke.  If you accidentally pressed `Esc`, press `C-g` to cancel the key combo.

## Readline shortcuts

These are the bread and butter.  The meat and potatoes.  When your basic paradigm revolves around single lines of text, you begin to appreciate the phrase *line editing*.  Mac OS X users might be familiar with these shortcuts already as most (not all) of them work pretty much everywhere you can type text.

This is by no means a comprehensive list, but just some of my favorites.

| Key combo | What it does |
| --------- | ------------ |
| `C-b` | Move back one character |
| `C-f` | Move forward one character |
| `M-b` or `M-left` | Move back one word |
| `M-f` or `M-right` | Move forward one word |
| `C-a` | Move to the start of the line |
| `C-e` | Move to the end of the line |
| `C-k` | Cut everything to the right of the cursor |
| `C-u` | Cut everything to the left of the cursor |
| `C-a C-k` | Cut the entire line |
| `C-e C-u` | Cut the entire line [(there is more than one way to do it)](http://www.c2.com/cgi/wiki?ThereIsMoreThanOneWayToDoIt) |
| `C-w` | Cut one word to the left of the cursor.  If multiple words are cut, they all get pasted together. |
| `M-d` | Cut one word to the right of the cursor. If multiple words are cut, they all get pasted together. |
| `C-y` | Paste the last thing you cut (this is *not* the OS's clipboard) |
| `C-t` | Twiddle the previous character forwards |
| `C-i` | That's the Tab key |
| `C-[` | That's the Escape key |
| `C-h` | That's the Backspace key (backwards) |
| `C-d` | That's the Delete key (forwards) |
| `C-j` | That's the Enter key |
| `C-m` | That's the Return key  |
| `C-l` | Clear the console.  I'm always using this, especially `C-p C-l`. |
| `C-p` | Scroll up to the previous command |
| `C-n` | Scroll back down to your current command input |
| `C-r` | Search previous commands that start with `text you enter`.  This is my favorite.  Repeat `C-r` to keep searching.  Press `C-g` to quit the search.  Press `Enter` to run the previous command as-is or `C-a`/`C-e` to edit it. |
| `C-x C-e` | Open up an editor to edit your command |
| `C-x C-x` | Not quite sure what this one does, honestly.  Bounce back and forth in some inexplicable manner. |
| `C-c` | Interrupt / cancel the running process |
| `C-z` | Suspend the running process (more on that below) |
| `C-d` | Quit the shell.  Also the Delete key.  Only quits when the line is empty (so `C-a C-k C-d` always quits). |

## Jobs

Bash is a basic *job controller*.  You can fire up multiple processes all running concurrently within the same shell.  You can pause/play those processes as you like.  Let's run a silly little script just to demonstrate:

~~~bash
(while true; do echo 'Marco' && sleep 5; done)
~~~

Press `C-c` to cancel it.  Those parentheses are important!  They run the script in a *subshell*, which basically makes the entire one-liner run as a bona fide process instead of parsing the bash language in *this* shell.  Try this part without parentheses and you'll see what I mean.  Run it again, then press `C-z` to suspend it.

~~~bash
Marco
^Z
[1]+  Stopped                 ( while true; do
    echo 'Marco' && sleep 5;
done )
~~~

While that's stopped, let's run another script:

~~~bash
(while true; do echo 'Polo!' && sleep 2; done)
~~~

Let's suspend that with `C-z`, too.

~~~bash
Polo!
Polo!
^Z
[2]+  Stopped                 ( while true; do
    echo 'Polo!' && sleep 2;
done )
~~~

*Ominous*!  A wild `[2]` appears.  Let's run just *one* more script.

~~~bash
(while true; do echo 'Friday' && sleep 1; done)
~~~

We'll suspend that one, too, and we should get a `[3]`.

~~~bash
Friday
Friday
^Z
[3]+  Stopped                 ( while true; do
    echo 'Friday' && sleep 1;
done )
~~~

Now let's crack this thing wide open.  Run `jobs -l`.

~~~bash
[1]    897 Suspended: 18           ( while true; do
    echo 'Marco' && sleep 5;
done )
[2]-   917 Suspended: 18           ( while true; do
    echo 'Polo!' && sleep 2;
done )
[3]+   925 Suspended: 18           ( while true; do
    echo 'Friday' && sleep 1;
done )
~~~

Huzzah!  We're really hacking the Gibson now.  Let's bring one of these back to the foreground with `fg`.  (Which one will come back?)

~~~bash
( while true; do
    echo 'Friday' && sleep 1;
done )
Friday
Friday
^Z
[3]+  Stopped                 ( while true; do
    echo 'Friday' && sleep 1;
done )
~~~

It was Friday.  Friday came back.  Let's suspend it and check `jobs -l` again.

~~~bash
[1]    897 Suspended: 18           ( while true; do
    echo 'Marco' && sleep 5;
done )
[2]-   917 Suspended: 18           ( while true; do
    echo 'Polo!' && sleep 2;
done )
[3]+   925 Suspended: 18           ( while true; do
    echo 'Friday' && sleep 1;
done )
~~~

See that `+` next to the `[3]`?  `fg` will run whatever has the `+` next to it.  The `-` marks the next-in-line process, so it looks like it'll become the `+` if I kill that third job.

Let's kill that third job.  No more Friday.  We could kill it by its Process ID, conveniently listed as `925`, but that's too much typing.  We'll kill it by its *Job Number*, `%3`.  Run `kill %3`.

~~~bash
[3]+  Terminated: 15          ( while true; do
    echo 'Friday' && sleep 1;
done )
~~~

Terminated.  Check `jobs -l` again.

~~~bash
[1]-   897 Suspended: 18           ( while true; do
    echo 'Marco' && sleep 5;
done )
[2]+   917 Suspended: 18           ( while true; do
    echo 'Polo!' && sleep 2;
done )
~~~

Looks like the `+` and `-` moved down the line.  So `fg` will bring back Polo now. Let's not do that; instead, let's bring Marco back first.  Run `fg %1`.

~~~bash
( while true; do
    echo 'Marco' && sleep 5;
done )
Marco
Marco
^Z
[1]+  Stopped                 ( while true; do
    echo 'Marco' && sleep 5;
done )
~~~

Marco came back!  Suspend it and check `jobs -l`.

~~~bash
[1]+   897 Suspended: 18           ( while true; do
    echo 'Marco' && sleep 5;
done )
[2]-   917 Suspended: 18           ( while true; do
    echo 'Polo!' && sleep 2;
done )
~~~

Interesting.  The `+` is on Job 1 now, as it was the most recently suspended.  But wait, how do we run things concurrently?  That's what `bg` is for: running things in the background.  Let's run `bg %1`.

~~~bash
[1]+ ( while true; do
    echo 'Marco' && sleep 5;
done ) &
Marco
  dude@endor:~
$ Marco
Marco
~~~

We're getting Marcos all over the place.  Note that you can still type!  In fact, you can hit `C-c` and it won't cancel the process.  Let's run `jobs -l` amidst the noise:

~~~bash
[1]-   897 Running                 ( while true; do
    echo 'Marco' && sleep 5;
done ) &
[2]+   917 Suspended: 18           ( while true; do
    echo 'Polo!' && sleep 2;
done )
~~~

Run `bg %2` and we should get our expected output:

~~~bash
[2]+ ( while true; do
    echo 'Polo!' && sleep 2;
done ) &
  dude@endor:~
Polo!
$ Marco
Polo!
Polo!
Marco
Polo!
Polo!
Marco
Polo!
~~~

This is madness!  How do you stop them?  `C-c` and `C-z`, they do nothing!  Fortunately we can still bring them back into the foreground, then suspend them with `C-z`.  Run `fg %1` then `C-z`.  Then run `fg %2` and `C-z`.  Run `jobs -l`.

~~~bash
[1]-   897 Suspended: 18           ( while true; do
    echo 'Marco' && sleep 5;
done )
[2]+   917 Suspended: 18           ( while true; do
    echo 'Polo!' && sleep 2;
done )
~~~

Perhaps you noticed that when we run things in the background they show up with a trailing `&`:

~~~bash
[1]-   897 Running                 ( while true; do
    echo 'Marco' && sleep 5;
done ) &
~~~

It turns out you can skip the `C-z` / `bg` by putting a `&` after your command when you first run it.  Let's bring Friday back:

~~~bash
$ (while true; do echo 'Friday' && sleep 1; done) &
[3] 1048
Friday
  dude@endor:~
$ Friday
Friday
...
~~~

There's just one more useful trick, and that's `jobs -p %JOB`.  I just learned about this one recently.  If we just want the Process ID of Job #1, we could try parsing the output of `jobs -l`, but that's a pain.  Running `jobs -p %1` will give us exactly what we want.

~~~bash
$ jobs -p %1
897
~~~

You might wonder why that's useful.  `kill` seemed to work just fine on job numbers like `%3`, but not all commands will.  Let's see all the files that Job #1 has opened with `lsof` and `jobs -p %1`.

~~~bash
$ lsof -p $(jobs -p %1)
COMMAND PID USER   FD   TYPE DEVICE  SIZE/OFF    NODE NAME
bash    897 dude  cwd    DIR    1,2      3060  402246 /Users/dude
bash    897 dude  txt    REG    1,2    741596 4401824 /usr/local/Cellar/bash/4.3.33/bin/bash
bash    897 dude  txt    REG    1,2    432192  529497 /usr/local/Cellar/readline/6.3.8/lib/libreadline.6.3.dylib
bash    897 dude  txt    REG    1,2     76280  529492 /usr/local/Cellar/readline/6.3.8/lib/libhistory.6.3.dylib
bash    897 dude  txt    REG    1,2    622896 4765579 /usr/lib/dyld
bash    897 dude  txt    REG    1,2 385406082 6166029 /private/var/db/dyld/dyld_shared_cache_x86_64
bash    897 dude    0u   CHR   16,3   0t30313     633 /dev/ttys003
bash    897 dude    1u   CHR   16,3   0t30313     633 /dev/ttys003
bash    897 dude    2u   CHR   16,3   0t30313     633 /dev/ttys003
bash    897 dude  255u   CHR   16,3   0t30313     633 /dev/ttys003
~~~

To paste the output of the `jobs -p %1` command into the command we want to actually run, we nest it inside `$( dollar parens )`.

## Finis

That's all, folks.  Maybe later I'll talk about writing bash scripts.  Happy hacking.

[shellshock]: https://en.wikipedia.org/wiki/Shellshock_(software_bug)
