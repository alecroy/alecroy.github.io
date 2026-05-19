---
title: "Basic Idempotent Text Editing"
date: 2026-03-27T23:08:34Z
draft: true
---

Suppose you want to apply an edit to a textfile but you don't know if it's necessary.

Like suppose I want to "fix up" my server's `.ssh/authorized_keys` but I don't *want* to actually append my key block unless it's *already there*.

And suppose I want to do it programmatically.

I will note that there is some automated ways to do stuff like this for SSH, 

> ssh-keygen(1) also offers some basic automated editing for
~/.ssh/known_hosts including removing hosts matching a host name and
converting all host names to their hashed representations.

*and* you could probably be fine "just" appending to the file ... but I'm kind of brainstorming here how I'd do it in general.  Well, let's review some stuff, and write some Node.

Curiously, someone has written a [Rust parser for authorized_keys](https://docs.rs/authorized_keys/latest/authorized_keys/).

## .ssh/authorized_keys

It does look like `authorized_keys` is a one-line-per-entry kind of file.  I'm looking at `man 8 sshd`.  It says

> Public keys consist of the following space-separated fields: options, keytype, base64-encoded key, comment.  The options field is optional.  The supported key types are:
>
>     sk-ecdsa-sha2-nistp256@openssh.com
>     ecdsa-sha2-nistp256
>     ecdsa-sha2-nistp384
>     ecdsa-sha2-nistp521
>     sk-ssh-ed25519@openssh.com
>     ssh-ed25519
>     ssh-rsa
>
> The comment field is not used for anything (but may be convenient for the user to identify the key).

So the options allowing spaces within quotes is a little annoying, but fine.  At least it's singular lines (though I am curious about the multi-line scenario).

I guess I don't care what the options are, for the most part.  I don't really expect an option to include something that would look like `ssh-rsa something-that-looks-keylike`, though a fully robust solution might ... tradeoffs.  Surely you wouldn't write something like that in a comment!?

At least the options are specifiied, like the key types, but there are like 20 options.  Hmm.

At this point, the single-line nature seems worse than using a regex on YAML, but maybe that's the path to suffering; I don't know.

There's definitely a doing a decode-work-encode transform cycle (like an SVD) to like 

1. parse config from JSON to JavaScript object
1. modify the object with normal code
1. write JavaScript object to JSON

That's the dream, I guess.  Same for YAML.

But what about these textfile configurations, like `authorized_keys` ?

I think I'd have to turn to Ohm/OMeta, and try writing a PEG.  The [Ohm editor](https://ohmjs.org/editor/) is and looks amazing.  I'm a bit tired from traveling to do it today, though ...


