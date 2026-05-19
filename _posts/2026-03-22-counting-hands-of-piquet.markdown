---
title: "Counting Hands of Piquet"
date: 2026-03-22T15:33:46Z
params:
  math: true
---

One counting problem I sometimes think about is: how many hands of Piquet *are there* ?

It seems hard to say, since there are different ways to count it:

1. out of a deck of 32 cards, you're basically given 16: $32 \text{ choose } 16 = 601,080,390$

2. out of those 16, you set aside 4: $16 \text{ choose } 4 = 1,820$ (though you would tend to set aside the lowest 4, so you hardly explore all of these)

3. so you basically get 12 cards of the 32: $32 \text{ choose } 12 = 225,792,840$ (though setting things aside makes this not quite right)

4. suits are equivalent if they have the same exact cards, but how much can that reduce things?  if I get like 2 clubs and 2 spades, well there are $8 \text{ choose } 2 = 28$ ways that could happen, so only 1 of those 28 would "collapse" possibilities

5. but lower cards hardly matter!  For each suit, the topmost cards certainly matter more than the others: A K Q 9 is much different from A J 10 9.  Though even if you lack the Ace, having K Q J is still pretty important.  So maybe focusing on the top 3 or 4 cards from each suit would help

6. so of the top 12 cards in the deck, you could draw them all, or none, anywhere in between.  $2^{12} = 4,096$, reduced further by suit-equivalence

7. same for the top 16 cards, though, if you count the 4 cards set aside: $2^{16} = 65,536$

8. if you only care about Aces & Kings, there's only $256$ possibilities, and now reducing by suit equivalence could really bring that down

So I think if you're trying to model the game by dealt hand, your model would probably have to distinguish *thousands to millions* of different ways the cards could be dealt, depending on how finely you want to distinguish one hand from another.

I think the ~1,000 mark is pretty good for a game to get to.  You could play about ~1,000 hands without feeling like you're playing the same hand twice.

## Aces & Kings &c &c

So let's try reducing $256$ by suit equivalence.

For example, ♣️A ♦️A ♠️K ♥️K is basically the same as ♣️K ♦️K ♠️A ♥️A.  Likewise, ♣️AK ♦️AK is the same as ♠️AK ♥️AK.

I guess I could normalize the 8-bit numbers by 2-bit chunks (sorting the chunks).  I guess I could do an arbitrary number of top cards:

~~~java
private int count(int topCards) {
    final var H = new HashSet<Integer>();

    int mask = 0;
    for (int bit = 1, c = 0; c < topCards; bit <<= 1, c++) {
        mask |= bit;
    }

    for (int i = 0; i < (1 << (4 * topCards)); i++) {

        final var chunks = new int[] {
                ((i) & mask),
                ((i >> topCards) & mask),
                ((i >> (2 * topCards)) & mask),
                ((i >> (3 * topCards)) & mask),
        };

        Arrays.sort(chunks);

        final var norm = (
                (chunks[0] << (3 * topCards))
                        | (chunks[1] << (2 * topCards))
                        | (chunks[2] << topCards)
                        | (chunks[3])
        );

        H.add(norm);
    }

    return H.size();
}
~~~

yields

~~~
top 1 cards => 5
top 2 cards => 35
top 3 cards => 330
top 4 cards => 3876
top 5 cards => 52360
top 6 cards => 766480
~~~

So I think it's fair to say there are about four thousand to a million hands of Piquet, depending on how precisely you want to model things.
