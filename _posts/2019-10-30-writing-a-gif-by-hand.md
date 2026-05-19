---
layout: post
title: "Writing a GIF by Hand"
date: 2019-10-30T17:50:00-04:00
categories: Blogging
---

Suppose you've got a blog.

This blog has a favicon, and for some awesome reason you want to inline this gif right into the `<head>` of every single page on this blog.
<!--more-->

For some reason, you settle on GIF to be your format.  We're rolling the dice, on graphical peanut butter[^0].

You want this GIF to be small.  Very small.

You can settle for near-maximum constraints: 2 solid colors, adjacent rectangles of red & white.  Shameless.  We'll call it minimalism, or Bauhaus or something.  Industrial, like Rammstein (right?).

How do we write this down?

https://en.wikipedia.org/wiki/GIF

## First, an example

Fortunately, I already have a favicon on disk that fits this profile.  Suppose I forgot how I generated it.  I think it came out of GIMP, but I'm not sure.  Whatever it was, that method sucked; I wasn't writing it by hand!  Way cooler to just *.. __write__* an image.  Artists *draw* pictures, so maybe this software engineer can *write* one.  It'll have to be a Rothko.  It looks like:

      $  od -t x1 favicon.gif 
    0000000    47  49  46  38  39  61  10  00  10  00  a1  00  00  b3  1b  1b
    0000020    f7  f7  f7  b3  1b  1b  b3  1b  1b  21  f9  04  01  0a  00  02
    0000040    00  2c  00  00  00  00  10  00  10  00  00  02  1f  84  1f  a9
    0000060    7b  b8  df  cc  83  71  b2  6a  53  04  59  e3  bc  75  41  d8
    0000100    91  e0  67  99  29  3a  a9  2d  4b  35  62  01  00  3b        
    0000116

Pretty small already!  But I'm not sure it's the best.  We're loading this on *every* page, so there's no excuse for waste.

Let's break it down.

    47  49  46  38  39  61  -- GIF89a
    10  00  -- width = 16
    10  00  -- height = 16
    a1  -- GCT = 1 (enabled)
        -- Color Resolution = 2 (3 bits per primary color?? not sure if used)
        -- Sort = 0 (GCT unordered)
        -- GCT Size = 1 (2∙2^1 = 4 entries)
    00  -- Background Color Index
    00  -- Pixel Aspect Ratio
    b3  1b  1b  -- GCT color #0 R G B
    f7  f7  f7  -- GCT color #1 R G B
    b3  1b  1b  -- GCT color #2 R G B
    b3  1b  1b  -- GCT color #3 R G B
    21  f9  04  01  0a  00  02  00  -- unknown?
    2c  -- Image Separator
    00  00  -- Image Left Position
    00  00  -- Image Top Position
    10  00  -- Image Width
    10  00  -- Image Height
    00  -- LCT, Interlace, Sort, Reserved, LCT Size = 0
    02  -- LZW Minimum Code Size = 2 bits (could it be 1?)
    1f  -- Sub-block of 31 bytes follows
    84  1f  a9  7b  b8  df  cc  83
    71  b2  6a  53  04  59  e3  bc
    75  41  d8  91  e0  67  99  29
    3a  a9  2d  4b  35  62  01  -- Image Data
    00  -- Block terminator
    3b  -- Trailer  

So we've got a bloated GCT we can trim down, and 31 bytes of image data.

We have to decode by hand, I guess.  Shit sucks, but in computers there's often no one to hold your hand.

I'm just going to quote the [source](https://www.w3.org/Graphics/GIF/spec-gif89a.txt):

## *Appendix F. Variable-Length-Code LZW Compression.*

> *The Variable-Length-Code LZW Compression is a variation of the Lempel-Ziv
Compression algorithm in which variable-length codes are used to replace
patterns detected in the original data. The algorithm uses a code or
translation table constructed from the patterns encountered in the original
data; each new pattern is entered into the table and its index is used to
replace it in the compressed stream.*

Sounds reasonable.  So how do we build this table, and how/where is it written down?

> *The compressor takes the data from the input stream and builds a code or
translation table with the patterns as it encounters them; each new pattern is
entered into the code table and its index is added to the output stream; when a
pattern is encountered which had been detected since the last code table
refresh, its index from the code table is put on the output stream, thus
achieving the data compression.*

Well, it's only "compression" if the index is a smaller number than the pattern itself, but I'm sure we'll get to that, so yes..

> *The expander takes input from the compressed
data stream and builds the code or translation table from it; as the compressed
data stream is processed, codes are used to index into the code table and the
corresponding data is put on the decompressed output stream, thus achieving
data decompression.  The details of the algorithm are explained below.  The
Variable-Length-Code aspect of the algorithm is based on an initial code size
(LZW-initial code size), which specifies the initial number of bits used for
the compression codes.  When the number of patterns detected by the compressor
in the input stream exceeds the number of patterns encodable with the current
number of bits, the number of bits per LZW code is increased by one.*

I don't have the original input stream (I mean, I can imagine it's a bunch of square containing rectangles of 0s and 1s, referring to my 2 colors).

If it read the image row-by-row, and it's a 16x16 image, it'd really be storing 5 or 6 pixels of one color, then 10 or 11 of the other.  So the table should ideally have just 2 entries, one for a short run of red and another for a longer run of white.  I'm not sure that's what I'll find, but that's what I'd try to write.

    84  1f  a9  7b  b8  df  cc  83
    71  b2  6a  53  04  59  e3  bc
    75  41  d8  91  e0  67  99  29
    3a  a9  2d  4b  35  62  01  -- Image Data

First, to answer my question of Can The LZW Minimum Code Size = 1?  It seems maybe not:

> *Because of some algorithmic constraints however, black & white images which have one color bit must be indicated as having a code size of 2.*

Ugggggggh.  Let's do some more research.  Like [What's In a GIF?](http://www.matthewflickinger.com/lab/whatsinagif/bits_and_bytes.asp)!

Aha, this clarifies the unknown line from before:

    21  f9  04  01  0a  00  02  00  -- unknown?

This is a Graphics Control Extension: (isn't this optional?)

    21  -- Extension Introducer = 21
    f9  -- Graphic Control Label = f9
    04  -- Byte size
    01  -- Reserved = 0
        -- Disposal Method = 0
        -- User Input Flag = 0
        -- Transparent Color Flag = 1
    0a  00  -- Delay time
    02  -- Transparent Color Index
    00  -- Block terminator

This leaves the Image Data.

## Image Data

From before,

    84  1f  a9  7b  b8  df  cc  83
    71  b2  6a  53  04  59  e3  bc
    75  41  d8  91  e0  67  99  29
    3a  a9  2d  4b  35  62  01  -- Image Data

At this point, we have to decode in chunks smaller than bytes.  The characters coming off this stream will start off 3 bits wide (given LZW Minimum Code Size = 2, accounting for the clear code and EOI code on top), then it might grow to 4 bits, 5 bits, &c.

From what I understand, reading [danbbs.bk](http://web.archive.org/web/20050217131148/http://www.danbbs.dk/~dino/whirlgif/lzw.html), is that the GIF compression basically maintains a table of patterns.  This table starts with a pattern for each letter in the alphabet.  Compressing involves finding the shortest prefix-of-input not stored as a pattern, and add it to the "bottom" of the table.  The true bottom is for the clear code and terminator code, which start as #4 #5 but then move to #8 #9, #16, #17.  The output doesn't actually involve the pattern we just added to the table; the weird thing about it is that we only add patterns to the table we're matching separately.  That is, matching *ABB* against an alphabet *A B C D*,

> *A* is in the table as #0
> *AB* is not in the table, store as pattern #4 (clear/terminator now #8 #9)
> output not #4, but #0, then output-compress the remaining *BB* input stream
> *B* is in the table as #1
> *BB* is not in the table, store as pattern #5
> output #1
> output #1 for final B

It's pretty much why it's so easy to bootstrap the algorithm, without transmitting the table (other than the concept, in writing, like this).  This kind of thing is just perfect for Erlang bit-pattern-matching, but anyhow!

Let me paste the image data again, as binary:

> *10000100 00011111 10101001*
> *01111011 10111000 11011111 11001100 10000011 01110001*
> *10110010 01101010 01010011 00000100 01011001 11100011*
> *10111100 01110101 01000001 11011000 10010001 11100000*
> *01100111 10011001 00101001 00111010 10101001 00101101*
> *01001011 00110101 01100010 00000001 00000000 00111011*

Let me just take a lot of room and work bit by bit, 

| code # | output |
| ------ | ------ |
| #0     | 0 |
| #1 | 1 |
| #2 | 2 |
| #3 | 3 |
| #4 | clear (at first) |
| #5 | eof (at first) |
| more as we go.. | .. |

I think the new codes are added below the #4 #5 but I'm not entirely sure.  The rules of the LZW Game Show, as I understand it:

  1. You eat input as large as you recognize (keyed in table), and stop when you get a 2nd piece you wouldn't recognize if combined with the 1st.
  2. Your 1st input (sans 2nd piece) gets written down.  The whole input (1st & 2nd pieces together) gets keyed in the table as the next-free-key.
  3. The 2nd input becomes the 1st input, and we continue looking for a 2nd piece that will lead to an unrecognizable whole, or we are done and write down the 1st piece

Therefore what gets written are prefixes-we-know of larger things we didn't know at the time.

## Entering the unknown

At this point, I have really no idea what's going on.  Apparently the bytes are read first-to-last, but bits inside are read right-to-left?  What's up with that?  And I'm not even 100% on that.  We'll keep trying things.

                        10 000 100
              0001 111 1
    1010 1001

Maybe #4 #0 #6 #7 #1 #9 #10?

I give up.  I'll figure this out later, y'all.

[^0]: And yes, it's pronounced like JIF, the peanut butter
