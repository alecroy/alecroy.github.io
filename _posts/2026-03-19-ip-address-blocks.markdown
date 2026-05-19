---
title: "IP Address Blocks"
date: 2026-03-19T14:07:41Z
---

I recently learned 127.0.1.1 is basically the same as 127.0.0.1.  It seemed strange I'd never seen it before.  I'm still not sure why you'd prefer one over the other.

LAN addresses are usually 192.168.  Other LANs sit on 10.  Docker seems to use 172.

So where are all of these written down?  Are there others?  Perhaps ... [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918) will tell me everything.  Or [RFC 5735](https://www.rfc-editor.org/rfc/rfc5735) ... there seem to be lots of RFCs about them.

## Specialized Address Blocks

This is most of them:

1. __0/8__ — 0.0.0.0–0.255.255.255 ("this" network)

1. __10/8__ — 10.0.0.0–10.255.255.255 (private network)

1. __127/8__ — 127.0.0.0–127.255.255.255 (this host)

1. __169.254/16__ — 169.254.0.0–169.254.255.255 (this local link)

1. __172.16/12__ — 172.16.0.0–172.31.255.255 (private network)

1. __192.0.2/24__ — 192.0.2.0–192.0.2.255 (example network, like `example.com`)

1. __192.168/16__ — 192.168.0.0–192.168.255.255

1. __198.51.100/24__ — 198.51.100.0–198.51.100.255 (another example network)

1. __203.0.113/24__ — 203.0.113.0–203.0.113.255 (yet *another* example network)

1. __224/4__ — IPv4 multicast (that's a big block)

1. __240/4__ — reserved

1. __255.255.255.255__ — limited broadcast

More on that last one: (let me just quote [RFC 922](https://www.rfc-editor.org/rfc/rfc922.html))

> The address 255.255.255.255 denotes a broadcast on a local hardware
>   network that must not be forwarded.  This address may be used, for
>   example, by hosts that do not know their network number and are
>   asking some server for it.
>
> Thus, a host on net 36, for example, may:
>
>      - broadcast to all of its immediate neighbors by using
>        255.255.255.255
>
>      - broadcast to all of net 36 by using 36.255.255.255
>
>   without knowing if the net is subnetted; if it is not, then both
>   addresses have the same effect. A robust application might try the
>   former address, and if no response is received, then try the latter.
>   See [1] for a discussion of such "expanding ring search" techniques.
>
>   If the use of "all ones" in a field of an IP address means
>   "broadcast", using "all zeros" could be viewed as meaning
>   "unspecified".  There is probably no reason for such addresses to
>   appear anywhere but as the source address of an ICMP Information
>   Request datagram.  However, as a notational convention, we refer to
>   networks (as opposed to hosts) by using addresses with zero fields.
>   For example, 36.0.0.0 means "network number 36" while 36.255.255.255
>   means "all hosts on network number 36".

Interesting!  Today I learned.  There is a lot about IPv4 I don't even know ...

It is strange the __127/8__ block is *just as large* as the __10/8__ block, but I guess somebody somewhere is making use of all of those loopback addresses.  At least, I hope so.
