---
title: "A QUIC Refresher"
date: 2026-03-14T07:26:57Z
---

HTTP has come a long way.  Apparently, HTTP/3 has been out for years!  I feel like I should know how it works.  I know very little about HTTP/2 (something about prefetching?), and basically nothing about HTTP/3, so I should at least try and catch up.

HTTP/3 runs on QUIC, which runs on UDP, which ... well, the point is to understand HTTP/3, I'm going to have to learn QUIC first.  I'm going to have to read through some RFCs, and try to understand what I can.

## TCP is dead, except not really

TCP has been due for an upgrade for a long time.  It's not a bad protocol!  It's been around, what, like 50 years?  So by Lindy's law or whatever it's called, it'll probably be around for another 50 years.

And that's fine.  It'll always be there as a fallback.  But let's focus on the progress.  On the bright side, building on UDP means everything built for a TCP+UDP world is still going to work.  I mean, it's not like they came up with some backwards-incompatible TCPv3 or whatever that routers and firewalls would drop.  That's nice.

Supposedly, QUIC will afford us something like:

1. sessions to avoid reconnection handshakes

1. fewer handshakes in general

1. TLS mandatory ?

1. multiple streams per connection

Let's look at the ~four RFCs:

1. [RFC 8999](https://www.rfc-editor.org/rfc/rfc8999) Version-Independent Properties of QUIC

1. [RFC 9368](https://www.rfc-editor.org/rfc/rfc9368) Compatible Version Negotiation for QUIC

1. [RFC 9000](https://www.rfc-editor.org/rfc/rfc9000) QUIC: A UDP-based Multiplexed and Secure Transport

1. [RFC 9001](https://www.rfc-editor.org/rfc/rfc9001) Using TLS to Secure QUIC

1. [RFC 9002](https://www.rfc-editor.org/rfc/rfc9002) Loss detection and congestion control

## RFC 8999 Version-Independent Properties of QUIC

Let's start with their definition:

> QUIC is a connection-oriented protocol between two endpoints. Those endpoints exchange UDP datagrams. These UDP datagrams contain QUIC packets. QUIC endpoints use QUIC packets to establish a QUIC connection, which is shared protocol state between those endpoints.

Okay.  They speak of:

1. IPv4 / IPv6 / IPv? flexibility: it's not IPv4- or IPv6-specific

1. QUIC may have many versions, so this document describes how versions are marked & negotiated

1. there are 2 kinds of packets, Long & Short, each of arbitrary length, though the versioning only shows up on the Long packets

1. Version Negotiation sounds like a downward negotiation, when an endpoint can't support the version listed in a Long header

That's about it?  It basically only gives a structure for each kind of packet header, but otherwise says nothing about any specific version, or how negotiation would work (which, I guess, is version-specific, after all).

Oh, they have a long list of incorrect assumptions.  Some of the things you might think that aren't true: (to be clear, the following things are false in that they're not necessarily true)

- Long packets are only exchanged during connection establishment
 
- the first packets exchanged are Long packets

- packet numbers increment by +1

- clients speak first

- only servers negotiate versions

- version negotiation necessarily results in a change of version

- connection IDs change infrequently

- 2 endpoints only have 1 connection between them at a time

- the values in the Version field are the same in both directions

- a packet with a given value in the Version field is using that version

Very interesting.  I guess it makes sense negotiation doesn't always need to happen right away.  The last point is kind of a headscratcher, but I guess it's just a field ...

Let's skip forward to RFC 9368, which supposedly updates 8999, but is like 5x as long ...

### RFC 9368 Compatible Version Negotiation for QUIC

> QUIC does not provide a complete version negotiation mechanism but instead only provides a way for the server to indicate that the version the client chose is unacceptable. ... Optionally, if the client's chosen version and the negotiated version share a compatible first flight format, the negotiation can take place without incurring an extra round trip.

This RFC's definitely more specific than the other one

> ... the "first flight" of packets refers to the set of packets the client creates and sends to initiate the connection before it has heard back from the server.
... the "client's Chosen Version" is the QUIC version of the connection's first flight.  The "Original Version" is the QUIC version of the very first packet the client sends to the server

Okay.  I guess I'm not clear when you'd have (client's Chosen Version != Original Version), but it'll probably be obvious.  Ah, so they say when a server basically fails a version and sends back a Version Negotiation Packet, the client then starts a *2nd connection* with a different version.  So each connection has a chosen version, but the overall sequence can span multiple connections.

Something about client disregarding VNPs when they include their chosen version (kinda bizarre, sure, but maybe that's what some downgrade attacks would look like).  VNPs are also disregarded when they contain incorrect connection IDs, okay.

Only servers send VNPs.

"Compatible" is kind of a tricky term.  Sometimes, a version can be upgraded to another, but sometimes not.  They say the compatibility is not necessarily symmetric.  They define it, though:

> If A and B are two distinct versions of QUIC, A is said to be "compatible" with B if it is possible to take a first flight of packets from version A and convert it into a first flight of packets from version B.

Backwards-compatibility is not guaranteed for any future versions, and it's okay for client & server to not even agree that their versions are compatible.  Cool.

Clients must be made aware of the Negotiated Version in some documented way.  It might not be the version in the first Long packet it receives.

Clients can opt-out of compatible version negotiation, at which point the server will perform incompatible version negotiation instead (the VNP, it seems).  In that case, the second connection might have different versions offered by the client.  They say the "second" connection is ... conceptual, and the "two" connection objects could be one and the same.  Fine.

Clients order their available versions by preference.  The chosen version must be included in the available versions (surely it'd come first, no?).  The server can disregard that preference.  Servers don't order their available versions, and they might offer none at all.

Lots of ways to get a version negotiation error, it seems.  The list of available versions must be scanned to make sure the server negotiated logically.  I should probably reread this section (err, whole RFC) to better understand how it defeats (some, but not all?) downgrade attacks.

## RFC 9000 QUIC: A UDP-based Multiplexed and Secure Transport

Wow, this is an *extremely long* RFC.  I don't know what I was expecting.  This could take me *weeks* to read through.  I am only going to be able to skim it today.  We're talking:

1. TLS handshakes

1. 0-round-trip-time client sends

1. packet authentication & encryption

1. streams, bidirectional and unidirectional

1. reliable delivery, congestion control, credit-based stream creation/sends

1. client-driven connection migration

1. connection termination through shutdowns, timeouts, &c

There seems to be at least some level of nesting, or conceptual hierarchy, something like:

1. Connections, establishment, migration, termination, error handling

1. Streams, stream states, flow control

1. Datagrams, Packets, frames, acknowledgement, reliable delivery

Let's just try to understand flow control, since it kind of was the *C* in *TCP*.  The receiver sets initial data size limits for both individual streams and an overall total across all streams (MAX_STREAM_DATA, MAX_DATA).  This is the "credit".  These numbers only go up.  A sender can't exceed these limits (without causing an error), but they can communicate that they're at either limit (DATA_BLOCKED, STREAM_DATA_BLOCKED); communicating this is optional, so the receiver shouldn't necessarily wait for them before considering increasing the windows.  Sending the flow control frames (that bump up credit) along with other frames reduces network load, but isn't strictly necessary.

Similar to the data size limits, there's a monotonically increasing *number of streams* as well.  So it's up to the receiving side to bump things up, and I guess congestion control to bump things down.

There's *lots* more to read about; I still haven't worked through even one basic handshake in terms of actual packets & datagrams & streams & connections &c &c &c.

## RFC 9001 Using TLS to Secure QUIC

This might be longer than RFC 9000; it would probably help if I already knew TLS, but I don't, so ... at least they narrow it down to TLS 1.3, so I don't have to worry about older versions.  Future versions of QUIC could use something other than TLS 1.3, though ...

Let me just look at this diagram and try to soak in the knowledge:

	Client                                                    Server
	======                                                    ======

	Get Handshake
	                     Initial ------------->
	Install tx 0-RTT keys
	                     0-RTT - - - - - - - ->

	                                              Handshake Received
	                                                   Get Handshake
	                     <------------- Initial
	                                           Install rx 0-RTT keys
	                                          Install Handshake keys
	                                                   Get Handshake
	                     <----------- Handshake
	                                           Install tx 1-RTT keys
	                     <- - - - - - - - 1-RTT

	Handshake Received (Initial)
	Install Handshake keys
	Handshake Received (Handshake)
	Get Handshake
	                     Handshake ----------->
	Handshake Complete
	Install 1-RTT keys
	                     1-RTT - - - - - - - ->

	                                              Handshake Received
	                                              Handshake Complete
	                                             Handshake Confirmed
	                                           Install rx 1-RTT keys
	                     <--------------- 1-RTT
	                           (HANDSHAKE_DONE)
	Handshake Confirmed

This is just one possible handshake, one possible ordering, far from the whole picture.  I think it's fine to learn this stuff last, maybe.

## RFC 9002 Loss detection and congestion control

Packet headers have an encryption level that indicates the packet number space.  Within that space, the packet sequence number lives.  Loss detection is done separately for each packet number space.  Packet numbers never repeat for a single connection, and they're sent in monotonically increasing order.  It's permitted to skip certain numbers.

> QUIC separates transmission order from delivery order: packet numbers indicate transmission order, and delivery order is determined by the stream offsets in STREAM frames.

Packets can contain multiple frames of multiple types.  Data & frames that need reliable delivery are either acknowledged or declared lost & resent.  All packets are acknowledged, one way or another, but those acks might only come after an "ack-eliciting packet".  The non-ack-eliciting packets don't matter when it comes to calculating round-trip-time samples (which makes sense).

> latest_rtt = ack_time - send_time_of_largest_acked

There's also a minimum RTT that can be occasionally reset after disruptive network events.  The latest & minimum RTTs go into calculating a *smoothed RTT*.  Endpoints should collect at least 3 packets beyond an unreceived packet before declaring it lost so as to avoid spurious retransmission.  There's also a timer based on the smoothed RTT.  This gets into like *tuning*.

When ack-eliciting packets aren't acknowledged when expected (calculated from the smoothed RTT &c), at least one more ack-eliciting packet must be sent as a probe, plus one or two probe datagrams can be sent (per packet number space) to allow the connection to recover from loss of tail packets.  If no new data can be sent, old data can be sent again.

Sender-side congestion control is similar to TCP NewReno (RFC 6582).  I wish I knew what that was ...

The initial congestion window should be at least 14,720 bytes.  That's interesting.  Congestion control states move between Slow Start, Recovery, & Congestion Avoidance.  The congestion window grows exponentially, then halves upon entering recovery (?), ideally ending up in congestion avoidance, where the window is controlled with Additive Increase Multiplicative Decrease.  That's probably mentioned in NewReno.

Persistent congestion is a condition it seems pretty reluctant to enter prematurely.  There have to be 2 ack-eliciting packets sent sufficiently far apart that are declared lost, plus no packets sent between them acknowledged (on any packet number space), all after an RTT sample has been established.  And finally, a later acknowledgement has to come through.  (I think I'm reading that right)

Endpoints can implement pacing as they choose.  The idea is that short-term bursts of packets can lead to short-term congestion and loss, but under-utilizing the congestion window is also bad.  Some sort of balancing act.

And that's about it, aside from all the parts I glossed over ...


## Well, then

There's a lot to it.  I don't expect most people will have read through these (then again, neither have I, really).  There will probably be 1,000-page books covering all of this (maybe there already are?).  I wonder how many lines of code a QUIC implementation would involve?  I'm thinking like on the order of 30,000.  Maybe more?  Probably more; I don't know if it can use an existing TLS implementation (or most of one) like off the shelf, or if it'd have to be integrated or what.

It seems like a really well-designed protocol!  Like if TCP lasted 50+ years, this one could last even longer.  Who knows?

