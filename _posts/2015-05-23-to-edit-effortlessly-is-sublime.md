---
layout: post
title: "To edit effortlessly is Sublime"
date: 2015-05-23 13:22
---

I use Sublime Text 2 quite a bit.  In this post, I'll talk about the few things I've learned about it.  Before I jump right in, let me bore you with some backstory.

### I was born on a Friday..

I used Notepad growing up.  Later I used Visual C++ 1.0 (not the Studio), [like so][vsc].  Dev-C++ was amazing when I found that, but I'd say GVim was the first real editor I used.  It was clearly built for *just editing*.  I found vim totally refreshing, and had a nice little `.vimrc` with leader keys and remaps and everything.  Soon after that, I switched over to the Dvorak keyboard layout and editing in vim became very unnatural (hjkl get separated, for instance).  I stuck to Eclipse for a while until one of my professors used Emacs in class one day.  I was pretty impressed to say the least, especially with the REPL interaction (via `tuareg-mode`).  Enamored, I scraped by on the bare minimum of keybindings (so much `C-g`) until I got comfortable (`C-h a` makes everything better).

I'll admit, I miss one thing about Emacs above all else: *indentation*.  You can smack the `TAB` key all day, and your source code only gets prettier (in most languages).  Most editors take a more literal approach (`\t`) that's less useful.

## Fast-forward

I use Sublime Text 2 every day.  I use the folder view.  I use multiple cursors.  I even tweaked a color scheme and made it mine.  It makes it all so *easy*.  I just barely knew how to kill rectangles in Emacs, and vi's VISUAL BLOCK mode wasn't much better.  Those tools work great when you can remember how to do everything, but I guess I can't.  I left it all behind in 2013 and bought Sublime Text.

I'm not an expert on Sublime; I'm just your average user.  I'll list most of my favorite hotkeys and share a couple config files.  I don't know the Windows/Linux-equivalent keys so you'll have to translate from OS X.  `C` is Caps/Control, `⌥` is Option, `⌘` is Command/Apple, `⇧` is Shift, and `⌫` is Delete.

| Key | What it does |
| --- | ------------ |
| `C-a/e` | Home/End |
| `C-n/p` | Next/previous line |
| `C-f/b` | Move forwards/backwards one character |
| `⌥-→/←` | Move forwards/backwards one word |
| `fn-⌥-⌫` | Delete next word |
| `⌥-⌫` | Delete previous word |
| `C-k` | Cut line(s) to the right of cursor |
| `C-y` | Paste deleted words/lines |
| `⌘-x/v` | Cut/paste the entire line (no selection) |
| `TAB`, `⇧-TAB` | Indent/unindent lines (from the beginning) |
| `C-⌘-↑/↓` | Move selected line(s) line up/down |
| `C-⇧-↑/↓` | Add another cursor to the line above/below |
| `⌘-/` | Comment/uncomment the line(s) |
| `⌘-d` | Add another cursor at the next copy of this selection |
| `⌘-⇧-p` | Open the Command Palette, to install stuff I guess |


That's it, I swear.  I hardly use anything else.  Call me crazy, call me lazy.  The one thing I *don't* do is open files from Sublime.  I *never* hit `⌘-o`; I don't really like file pickers, and even using the tab completion in `⌘-o ⌘-⇧-g` isn't as nice as plain old bash.  I almost always open files/folders from the command line.  If I'm already in a project directory, `st .` opens the entire folder.  More on that below.

I never hit the actual Control key, either; I use Caps Lock as Control.  [OS X has an option][osxkeys] to set that up in System Preferences, Windows uses [a simple registry key][winkeys], and Linux tends to [make it difficult][linuxkeys] (I'm looking at you, Ubuntu 14).

## Bash + Sublime

`st` is a bash alias I set up so that I can open files/folders from any terminal.  Opening up a new Sublime window is so easy I never worry about closing everything.  Here's the alias:

~~~bash
alias st='open -F -a /Applications/Sublime\ Text\ 2.app/ '
~~~

So for example, just to open my bash profile to copy that, I ran `st ~/.bash_profile` and got a fresh, new window with my profile (that's where my aliases live).  To not open files in new tabs in existing windows, you'll want `"open_files_in_new_window": true,` in your Preferences (`⌘-,`).  I don't know if Linux/Windows have an equivalent to `open`, but they should by now.  It's 2015.

I also use another helper, though much less often.  It's when I want to *create* the file first, then open it.  That alias above will barf if you try to open a non-existent file (I like it that way).

~~~bash
tst() {
    touch $@
    open -F -a /Applications/Sublime\ Text\ 2.app/ $@
}
~~~

Pretty straightforward: touch the file, then pop it open as before.  Nothing fancy.

## Styling

I change color schemes and fonts a lot, so I won't post anything specific.  Monaco 11 w/ Monokai is a great combo.  I also like to use [Courier Prime][cprime] 14 because it's got nice *italics*, which help a lot to highlight syntax.  I've been trying out [NK57 Monospace][nk57], too, at 12.2 but Sublime doesn't make it easy to use just the Semi-Condensed Book weight.  I ended up disabling all but 4 of the 60 weights in Font Book and that seems to work well enough for me.

There are so many good monospace fonts these days (Source Code Pro, Consolas, the ones I use, Droid Sans Mono, Fira Mono, ..) I can't possibly talk about them all.

There are probably 10 great color schemes for every great font, too, so I'll just point you to [ColorSublime][cs] and tell you to try [Tomorrow][tmw].


## Finis

That's all for this post.  Again, I'm not even really a power user, just your average editor.  I don't even use Sublime Text 3, just the 2.  You should probably check out more authoritative sources if you want to get the most out of Sublime.  Happy hacking.

[vsc]: http://www.betaarchive.co.uk/imageupload/1266131097.or.195465.png
[cprime]: http://quoteunquoteapps.com/courierprime/
[nk57]: http://typodermicfonts.com/nk57-monospace/
[osxkeys]: https://support.apple.com/kb/PH18422
[winkeys]: http://johnhaller.com/useful-stuff/disable-caps-lock
[linuxkeys]: http://askubuntu.com/questions/444714/how-do-i-swap-escape-and-caps-lock-in-14-04
[cs]: http://colorsublime.com/
[tmw]: http://colorsublime.com/?q=tomorrow
