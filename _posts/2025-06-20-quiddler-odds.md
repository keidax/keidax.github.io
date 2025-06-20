---
layout: post
title: "Measuring frustration in Quiddler, a word game"
date: 2025-06-20 12:45:00 -0400
tags: update
---

A favorite card game in my family is [Quiddler](https://www.playmonster.com/product/quiddler/).
The goal of the game is to spell one or more words with the cards in your hand, and score the most points with your words.
I recently became curious about a question: could I measure the probability of being able to form a valid hand on your first turn?

First, to briefly explain the rules:
A game of Quiddler starts with everyone being dealt three cards.
The number of cards increases each round.
On your turn, you can draw a new card, but you must discard.
Your goal is to "go out" by using **all** of your cards to spell one or more words.
Once someone at the table goes out, the rest of the players get one more turn.
Then they have to show their hands, whether they've formed words with all their cards or not.
Any cards that aren't part of a valid word are counted against your score for the round.

<img
  src="/assets/images/quiddler/quiddler.jpg"
  alt="A set of Quiddler cards on a wooden surface that spell KEIDAX">

> A random sample of Quiddler cards.

We have a long-standing house rule: you can't go out on your first turn.
This means each player gets at least two turns to recover from a bad start, like being dealt all consonants.
We came up with this rule after everyone became familiar with the frustration of the first player immediately throwing down a good hand while you're struggling to make sense of a bunch of Qs and Xs.
During a recent game, I wondered if this house rule could be slowing down play.
This prompted several questions:
1. As the game progresses and the number of cards increases, does it become more or less likely that you can go out right away?
2. What is the probability of being able to go out on your first turn?
3. How likely is it that the majority of players could all go out on their first turn?

---

To begin, I had to choose a word list.
Quiddler allows the players to choose their own dictionary, and the usual choice when we play is the _Official Scrabble Players Dictionary_.
I couldn't find an easily downloadable source of words for this dictionary, but I did learn about the [NASPA Word List](https://wikipedia.org/wiki/NASPA_Word_List), or NWL, which is used for tournament Scrabble play in the US.
It should be close enough for my purposes!
A direct download of the NWL requires an active NASPA membership, but the freely-available [Zyzzyva word study program](https://www.scrabbleplayers.org/w/Zyzzyva) includes multiple versions of the NWL in its data files.
I'll use the latest version, NWL2023.

With a word list in hand, I wrote a basic simulation of a Quiddler game.
It operates as follows:
1. For a round with `N` cards, randomly select `N+2` cards from the deck.
   The first `N` cards represent the initially dealt hand, the next card is the top of the discard pile, and the last card is drawn from the top of the deck.
2. Using the first `N+1` cards, check each possible combination of `N` cards.
   If one or more dictionary words can be made exactly with `N` cards, this counts as a valid hand.
   The process simulates the player taking the visible card at the top of the discard pile, then discarding to go out.
3. If the player hasn't gone out yet, remove the `N+1`th card from the set, and add the `N+2`th card.
   Repeat the process of checking each combination of `N` cards in the set.
   This simulates the player opting not to take from the discard pile, and drawing a face-down card instead.
4. Steps 1-3 are a single trial in the experiment.
   Repeat many more trials, and measure the number of successes and failures.

> This is a simplistic simulation. For instance, it doesn't account for the previous player being more likely to discard a difficult-to-use letter like J, Q, or Z.

The data and code used in this post are available at [github.com/keidax/quiddler-odds](https://github.com/keidax/quiddler-odds/).
Here's a display of the data:

<iframe
  src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQCwc0MZIQsZDH1Em2tH9fMefncCu9PcNekA3KpVa3ujT9zFCdCXjZczUUm3EG8xUY9EL-YfrULqnc1/pubchart?oid=87896930&amp;format=interactive"
  style="width:100%; height:371px; border:none;"
></iframe>

<!-- The height of 371px seems to be a static size of the embedded chart, I'm not if or how it can be changed. -->

With some initial results in hand, I began to grow suspicious.
This answers my first question: it's easier to go out when you have more cards.
But after 10,000 trials, it appeared that the chance of being able to go out on your first Quiddler hand of 3 cards was a whopping 85%.
This didn't feel right to me --- I knew from experience that more than 1 out of 5 hands is unplayable at first.
I took another look at the simulation, and realized what was going on: with access to the complete dictionary, my program had an inhumanly perfect vocabulary.
This means it uses words like KAE ("a bird resembling a crow"), ZAX ("a tool for cutting roof slates"), and OE ("a whirlwind off the Faeroe islands").
Maybe world-class Scrabble players could pull this off, but not most folks.

It was time to search for a new word list.

---

After some digging, I came across [Google's Trillion Word Corpus](https://research.google/blog/all-our-n-gram-are-belong-to-you/).
This seemed promising: I could create a dictionary based on how frequently words were used, thus more accurately representing the average person's working vocabulary.
Several processed versions of this dataset are available at [norvig.com/ngrams](https://norvig.com/ngrams/).
I used the `count_1w.txt` data file, which represents the 1/3 million most frequent words on the Internet (as of 2006).
This data already comes sorted by frequency, and filtering out invalid words (those that don't appear in NWL2023) is trivial.

Next I had to decide how large the dictionary should be.
I started looking into how many words the average person knows, and quickly realized [this is not a question with a single obvious answer](https://pmc.ncbi.nlm.nih.gov/articles/PMC4965448/).
It depends on the definition of a "word", on the definition of "knows", as well as factors like age and education.

This was quickly becoming more of a research project than I anticipated, so I went for an ad-hoc approach: I skimmed the word list until I reached an arbitrary cutoff point of increasingly unusual words, then rounded to a nice even number.
I ended up with a list of 20,000 mostly-common words, which I'm calling the "regular word" dictionary.

Granted, `regular_words.txt` has some obvious flaws for this project.
The choice of cutoff is haphazard; it doesn't really reflect anyone's actual vocabulary.
Since the original N-gram data was scraped from the Internet, it skews heavily towards technological vocabulary.
For example, according to the original dataset, ONLINE is the 80th most frequent word in English.
And the list still includes surprises like OE, which is apparently a few ranks more common than BEVERAGE.
(I suspect this is an artifact of how the data was gathered.)

---

Using the new list of regular words and rerunning the simulation, we see a measurable decrease in the number of hands that can go out immediately:

<iframe
  src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQCwc0MZIQsZDH1Em2tH9fMefncCu9PcNekA3KpVa3ujT9zFCdCXjZczUUm3EG8xUY9EL-YfrULqnc1/pubchart?oid=659220104&amp;format=interactive"
  style="width:100%; height:371px; border:none;"
></iframe>

To be honest, I don't really trust these numbers either.
The simulation still plays "perfectly" --- it will always find one or more dictionary matches if any valid words exist for the current set of cards.
That's just not how humans play games.
Part of the fun of Quiddler lies in scouring your brain for just the right set of words.
Nobody will perform that search perfectly, all the time.

Still, this gives an baseline on what plays are possible, and I'll consider this an answer to question #2.
It's time to move on to the last question: what are the odds that _most_ players can go out on their first turn in a round?

First, the estimated probabilities of going out from the "regular words" data:

| Number of cards | Estimated probability |
|---|---|
| 3 | 0.787 |
| 4 | 0.786 |
| 5 | 0.830 |
| 6 | 0.864 |
| 7 | 0.878 |
| 8 | 0.889 |
| 9 | 0.902 |
| 10 | 0.920 |

Next, let's establish some notation.
I'll use $$p(c)$$ to represent the probability of a player going out when dealt $$c$$ cards.
And I'll use $$P(g | n, c)$$ to mean the probability of $$g$$ players going out on the first turn when there are $$n$$ players dealt $$c$$ cards each.

Let's consider 3 players and 3 cards.
Each player has a 78.7% chance of going out on the first turn, since $$p(3) = 0.787$$.

$$P(0 | 3, 3) = (1 - 0.787)^3 \approx 0.010$$

$$P(1 | 3, 3) = (1 - 0.787)^2 \times 0.787 \times 3 \approx 0.107$$

$$P(2 | 3, 3) = (1 - 0.787) \times 0.787^2 \times 3 \approx 0.396$$

$$P(3 | 3, 3) = 0.787^3 \approx 0.487$$

Note the $$\times 3$$ appearing twice.
This represents the number of ways to select a single player in a group of 3 players.
We can generalize this formula. If $$n \choose g$$ is the number of ways to select $$g$$ players out of $$n$$ total players, then:

$$P(g | n, c) = (1 - p(c))^{n - g} \cdot p(c)^g \cdot {n \choose g}$$

> If this formula is unclear, I recommend looking up [binomial coefficients](https://wikipedia.org/wiki/Binomial_coefficient).

Next, we can calculate the [expected value](https://wikipedia.org/wiki/Expected_value) of the number of players going out.
I'll represent this as $$E[n, c]$$:

$$E[3, 3] \approx 0.010 \times 0 + 0.107 \times 1 + 0.396 \times 2 + 0.487 \times 3 = 2.36$$

What I'm actually interested in is the number of players who _can't_ go out on the first turn.
Let's call this the expected frustration:

$$E_{frustration}[n, c] = n - \sum_{g = 0}^{n} P(g | n, c) \cdot g$$

So $$E_{frustration}[3, 3] \approx 3 - 2.36 = 0.64$$.

Here's our final result: the expected frustration value for most rounds of Quiddler.

<table>
  <tr>
    <th>Number of cards </th>
    <th>3</th>
    <th>4</th>
    <th>5</th>
    <th>6</th>
    <th>7</th>
    <th>8</th>
    <th>9</th>
    <th>10</th>
  </tr>
  <tr>
    <th>3 players</th>
    <td>0.64</td>
    <td>0.64</td>
    <td>0.51</td>
    <td>0.41</td>
    <td>0.37</td>
    <td>0.33</td>
    <td>0.29</td>
    <td>0.24</td>
  </tr>
  <tr>
    <th>4 players</th>
    <td>0.85</td>
    <td>0.86</td>
    <td>0.68</td>
    <td>0.54</td>
    <td>0.49</td>
    <td>0.44</td>
    <td>0.39</td>
    <td>0.32</td>
  </tr>
  <tr>
    <th>5 players</th>
    <td>1.06</td>
    <td>1.07</td>
    <td>0.85</td>
    <td>0.68</td>
    <td>0.61</td>
    <td>0.55</td>
    <td>0.49</td>
    <td>0.40</td>
  </tr>
  <tr>
    <th>6 players</th>
    <td>1.28</td>
    <td>1.28</td>
    <td>1.02</td>
    <td>0.82</td>
    <td>0.73</td>
    <td>0.67</td>
    <td>0.59</td>
    <td>0.48</td>
  </tr>
  <tr>
    <th>7 players</th>
    <td>1.49</td>
    <td>1.50</td>
    <td>1.19</td>
    <td>0.95</td>
    <td>0.85</td>
    <td>0.78</td>
    <td>0.69</td>
    <td>0.56</td>
  </tr>
  <tr>
    <th>8 players</th>
    <td>1.70</td>
    <td>1.71</td>
    <td>1.36</td>
    <td>1.09</td>
    <td>0.98</td>
    <td>0.89</td>
    <td>0.78</td>
    <td>0.64</td>
  </tr>
</table>

What does this tell us in the end?

I think the "regular words" data and the simulation process represent a better-than-average vocabulary, so I'm cautious about drawing conclusions from the specific numbers.
But it's clear that the frustration value trends down each round.
For anyone playing Quiddler with a similar house rule, then I would offer a modification:
Only limit when players can go out during the first four rounds of the game.
Hopefully this reduces the frustration of an early bad hand while keeping the game moving along.

Happy playing! <span class="emoji">🃏</span>

---
