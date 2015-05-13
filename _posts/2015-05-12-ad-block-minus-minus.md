---
layout: post
date: 2015-05-12 18:49
title: "Ad block, minus minus"
---

Recently I updated my `/etc/hosts` file.  I don't currently use any ad blocking software.  The last one I used was Ghostery, which seemed to work well.  I've heard great things about the [Privacy Badger](https://www.eff.org/privacybadger), though I haven't downloaded it; it's not really so much an ad blocker but a *privacy tool*.  Anyhow.

My `/etc/hosts` file is 30,731 lines long.  72 lines are blank, 2,089 lines are comments, and 5 refer to localhost.  That leaves ~28,500 foreign hosts, all redirected to either [`127.0.0.1`](https://en.wikipedia.org/wiki/Localhost) or [`0.0.0.0`](https://en.wikipedia.org/wiki/0.0.0.0).  Some hosts might appear more than once, so it's a rough number.

It's not a perfect ad blocking solution.  I still see ads occasionally, but most seem to be gone.  It's not just about ads, though: I don't need my machine making useless connections to at best analytics and at worst malware sites.  It wastes bandwidth and energy I'd rather not use.  Plus, it's such an easy modification, I don't see any reason not to set it up.  Perhaps it will cause an issue using some sites.  If it does, I'll just inspect which HTTP requests are failing and rerun the tool, excluding those domains.  This hasn't happened yet.

Here's how I set it up.  It's pretty easy:

1.  Clone [StevenBlack's excellent repository](https://github.com/StevenBlack/hosts) from GitHub.  It's a large repository (for a bunch of textfiles), so clone it on a good Internet connection and/or be patient.

    ~~~bash
    git clone https://github.com/StevenBlack/hosts.git
    Cloning into 'hosts'...
    remote: Counting objects: 567, done.
    remote: Total 567 (delta 0), reused 0 (delta 0), pack-reused 567
    Receiving objects: 100% (567/567), 21.69 MiB | 312.00 KiB/s, done.
    Resolving deltas: 100% (235/235), done.
    Checking connectivity... done.

    cd ./hosts/
    ~~~

1.  Back up your original `/etc/hosts` file.  *(optional)*

    You might want to keep a copy of your original hosts file around, for old time's sake.

    ~~~bash
    cp /etc/hosts ~/hosts.original
    ~~~

1.  Run the tool.

    ~~~
    python updateHostsFile.py 
    Do you want to update all data sources? [Y/n] y
    Updating source malwaredomainlist.com from http://www.malwaredomainlist.com/hostslist/hosts.txt
    Updating source mvps.org from http://winhelp2002.mvps.org/hosts.txt
    Updating source someonewhocares.org from http://someonewhocares.org/hosts/hosts
    Updating source StevenBlack from https://raw.github.com/StevenBlack/hosts/master/data/StevenBlack/hosts
    Updating source yoyo.org from http://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&mimetype=plaintext&useip=0.0.0.0
    Do you want to exclude any domains?
    For example, hulu.com video streaming must be able to access its tracking and ad servers in order to play video. [Y/n] n
    OK, we won't exclude any domains.
    ==>::1 localhost<==
    ==>::1 localhost<==
    Success! Your shiny new hosts file has been prepared.
    It contains 25845 unique entries.
    Do you want to replace your existing hosts file with the newly generated file? [Y/n] y
    Moving the file requires administrative privileges. You might need to enter your password.
    Password:
    Flushing the DNS Cache to utilize new hosts file...
    No matching processes were found
    Flushing the DNS Cache failed.
    ~~~

1.  Don't worry about that last 'failed' part.  Flush the DNS cache by hand according to `readme.md`.  (OS X below, see the `readme.md` for Windows/Linux)

    ~~~bash
    dscacheutil -flushcache
    ~~~

1.  Done!  You can check that it blocks any/all domains in the file:

    ~~~bash
    ping adcore.ru
    PING adcore.ru (0.0.0.0): 56 data bytes
    ping: sendto: No route to host
    ~~~

    ~~~bash
    curl -v 'http://adcore.ru/'
    * Hostname was NOT found in DNS cache
    *   Trying 0.0.0.0...
    * connect to 0.0.0.0 port 80 failed: Connection refused
    * Failed to connect to adcore.ru port 80: Connection refused
    * Closing connection 0
    ~~~

That's all there is to it.  Happy hacking.
