---
title: "The Uncrushable - how tanks get attacked"
slug: "the-uncrushable-tank"
date: 2026-10-15
lastmod: 2026-10-15
description: ""
tags: []
math: true
draft: true
---

Have you heard about a tank in WoW being uncrushable or uncrittable? Maybe you saw some magic 102.4% floating around or some rather cryptic statements about probabilities that don't seem to add up (literally). I certainly have been left puzzled by the character sheet many times and, if anything, knowing a little about probability makes it worse. So this is my attempt to clear up some of this confusion, mostly for myself, hopefully for you too.


> **The short version.**
> - A swing ends in exactly one of seven outcomes. The only rule is that their probabilities add up to one.
> - WoW turns your stats into probabilities with one roll on a ruler from 0 to 100, cut into intervals in a fixed order. Stats add, tooltips are exact, and order of equipping is irrelevant.
> - The price is that the intervals can ask for more than the ruler has. Blizzard's rule is simple. The last interval pays, and when it is gone, the one above it.
> - A raid boss adds a crushing-blow interval and shaves 0.6 off each of your four. Cover the ruler and crush is gone. That is 102.4 on the sheet, and it is a cliff, not a slope.

## How attacks work

Imagine a game where each attack has a perfectly predictable (deterministic) outcome. Then you could tell before you engage whether you or your opponent is going to die. Sounds silly? A game like chess actually works that way, as a capture always succeeds. Games like Risk and D&D, on the other hand, add an element of chance for some extra spice. You cannot know for sure how each encounter will end and this places us in the realm of probabilities.

Every attack in WoW leads to one outcome chosen from a set of possibilities. For an attack on a tank, these possibilities usually are:

1. **Miss:** the swing fails, no damage
2. **Dodge:** you get out of the way, no damage
3. **Parry:** you deflect a hit, no damage
4. **Block:** absorbed by shield, reduced damage
5. **Crit:** a serious strike, double damage
6. **Crush:** a bad blow not as bad as crit, 150% damage, only from higher level bosses
7. **Ordinary:** normal damage

Randomness enters twice. The ordinary damage is randomly drawn from a range of possibilities and then, independently from that, one of the above outcomes is randomly picked. This leads to no damage, reduced damage, increased damage, or simply the ordinary damage as drawn. The damage draw is simple with a single number picked at random. The other one is where it gets confusing, and that is what I focus on here.

## Probabilities

The seven outcomes above are the sample space of a swing. Each possibility, say a dodge, is an exclusive outcome, meaning there can be only one, and the set is exhaustive, meaning it must be exactly one of those. Each of them can then be assigned a number reflecting its likelihood, and if they are positive and sum to one, they are probabilities. That they sum to one is the only rule the game is bound by. Everything else, including how a number on the character sheet becomes a probability, is a choice Blizzard made.

Of course, we could just uniformly pick an outcome and be done with it, like rolling a seven-sided die, but then there would be no role for gear, buffs, and talents to influence any of this. Instead, each of them is assigned a probability that reflects the stats from the character sheet. Where it gets interesting is how modifiers, like +1% dodge chance, actually affect each probability. Why can we not just add 1% to dodge? The numbers still need to add up to one, so some other probabilities need to give. Which numbers and by how much are the questions that...make WoW feel like WoW.

One way to look at this is as a sequence of outcomes. First, miss or not miss. If the attack does not miss, then dodge or not dodge. And so on, until we reach the ordinary damage, which applies if all others did not. Then if you pick up a Mark of Tyranny, the trinket from the Drakkisath quest in Upper Blackrock Spire, which says "Increases your chance to dodge an attack by 1%", a natural way to reflect that is to add one percentage point to the dodge outcome (8% becomes 9%), leaving the no-dodge outcome a 91% probability. Notice, however, how dodge is only determined after miss. This means the actual probability of getting a dodge outcome would not increase by 1 percentage point since this only applies IF the opponent does not miss. What we actually modified this way is the conditional probability, the chance of a dodge conditional on not missing. So if not missing has a chance of 95%, then this adjustment only increases the dodge chance overall by $0.95 \times 1 = 0.95$ percentage points. Not missing remains unaffected, while all outcomes further down the order (parry, block, and so on) have a lower probability. It does the job and yet a tooltip that only gives you the conditional probability seems undesirable. The game could, of course, calculate the true effect given the current miss probability, but then the tooltip of the same item would differ across players.
With the sheet reading 5% miss, 8% dodge, 5% parry and 5% block, the sequence turns those into the following shares of all swings, before and after the trinket:

| Outcome | Coin | Before | After +1% dodge | Change |
| --- | --- | --- | --- | --- |
| Miss | 5% | 5.00 | 5.00 | 0 |
| Dodge | 8% → 9% | 7.60 | 8.55 | +0.95 |
| Parry | 5% | 4.37 | 4.32 | −0.05 |
| Block | 5% | 4.15 | 4.11 | −0.04 |
| Everything else | remainder | 78.88 | 78.02 | −0.86 |

The trinket's one point of dodge arrives as 0.95, and every outcome after dodge in the sequence pays for it, in proportion to its size.


A more agreeable alternative is best illustrated by dropping the sequential perspective and instead looking at the outcome as a single draw. Consider a ruler with numbers from 0-100. Divide that ruler into several intervals whose length reflects the intended probability. With the numbers from before, miss gets 0-5 for its 5% chance, dodge gets 5-13 for its 8%, parry 13-18, block 18-23, and everything else 23-100. Each outcome is an interval. To get our random outcome, we then draw a number between 0 and 100 (and unlike a loot roll this doesn't have to be an integer), and the interval the number lands in is the final outcome.

So let's look again at the increased dodge chance. We need to lengthen the interval that corresponds to dodge by 1. Since our ruler has a fixed length of 100 numbers, however, we need to shrink some intervals, i.e., we need to normalise everything so we still get probabilities that add up to 1. With dodge at 5-13, we could simply add 1 to each cut-off from 13 onwards. We then get a total length of 101. We can then multiply each cut-off by 100/101 to make sure the total is right. This rescales each interval proportional to their length (i.e., longer intervals are shortened by more in absolute terms). Dodge asked for 9 of 101 and gets $9 \times 100/101 = 8.91$, so the trinket delivered 0.91 of its point, and every other interval gave up about 1% of itself. That is even less than the 0.95 the sequence delivered. This, however, has the same problem as before - the tooltip does not reflect final outcomes.

You might now object and suggest we should just rescale all the intervals that the trinket hasn't touched. More specifically, we shrink each untouched interval by the same factor, so that together they fit into whatever the trinket left us with. Dodge keeps its full 9. The other intervals add up to 92 and have to fit into 91, so each is multiplied by 91/92. Miss, for instance, goes from 5 to $5 \times 91/92 = 4.95$, and the ordinary-damage interval from 77 to 76.16. Now the tooltip is exact and the trinket delivered its full point.

This sounds great. There is a subtle problem, though. Say you equip a ring with +3% parry. We can then repeat the procedure so that item also keeps its full point and the untouched intervals shrink to make room. It turns out that the order in which we equipped the items, and thus computed the probabilities, now matters:

| Order | Dodge | Parry | Miss | Block | Everything else |
| --- | --- | --- | --- | --- | --- |
| Trinket first, ring second | 8.72 | 7.95 | 4.79 | 4.79 | 73.76 |
| Ring first, trinket second | 8.75 | 7.91 | 4.79 | 4.79 | 73.76 |

The difference is small here and yet this is a serious problem. Whatever the sheet shows isn't a function of what you are wearing but of the sequence in which you equipped it. It's easy to see why WoW's designers didn't go down that path.


What they did instead is much simpler. When you increase the length of one interval, you cut that same length from the last interval that goes up to 100. So in the case of our example, the ordinary damage one. All other intervals remain unchanged. Unless we completely push that last interval off the edge, in which case we move to the one above it in the list.

With our numbers, the trinket turns dodge from 5-13 into 5-14. Every cut-off after it moves up by one, so parry is now 14-19, block 19-24, and the ordinary damage interval runs from 24 to 100 instead of 23. Dodge got its full point, miss, parry and block are untouched, and ordinary damage paid for all of it. Add the ring and parry grows by three, from 14-19 to 14-22, block slides to 22-27, and ordinary damage shrinks to 27-100. Put the ring on first and the trinket second and you land in exactly the same place:

| Outcome | Sheet | Interval after both items | Share of swings |
| --- | --- | --- | --- |
| Miss | 5% | 0-5 | 5 |
| Dodge | 8% + 1% | 5-14 | 9 |
| Parry | 5% + 3% | 14-22 | 8 |
| Block | 5% | 22-27 | 5 |
| Everything else | remainder | 27-100 | 73 |

Every tooltip is exact, the sheet adds up, and the order you clicked things in is irrelevant.

All four rules on one ruler. Any way of turning the sheet into probabilities is a set of intervals on the same 0-100 line, the sequential coins included, so they can be drawn side by side. Tick the items and swap the order:

<iframe src="rules.html" width="100%" height="460" loading="lazy" title="Four rules for fitting a point of dodge onto the ruler: sequential coins, scale everything, keep the point and shrink the rest, last interval pays"></iframe>

The only thing that can go wrong is running out of ruler, and that is what the rest of this post is about.










## Running out of ruler

- Everything so far assumed the intervals fit. Nothing guarantees that. The four avoidance intervals are set by gear and buffs, independently of each other, and no item checks what the others add up to.
- Enter the boss. So far the attacker was a level-60 mob and the list had seven outcomes, but crush was empty. A raid boss is level 63. Three levels up does two things on the ruler: each of miss, dodge, parry and block loses 0.6 (0.2 per level per row, inferred), and a new 15-point interval appears, the crushing blow, between crit and ordinary damage.
- With the numbers from the table above, the boss sees 4.4 miss, 8.4 dodge, 7.4 parry, 4.4 block, then 5.6 crit, 15 crush, and ordinary damage gets what is left. Same gear, same sheet, a worse ruler.
- The ruler, live. Presets left to right: dungeon gear vs a level-60 mob, the same gear vs level 63, a raid tank with Shield Block down, Shield Block up a hair short, one point over the edge.

<iframe src="ruler.html" width="100%" height="680" loading="lazy" title="Attack table ruler: sliders for miss, dodge, parry and block, a boss level toggle and three rules for resolving the roll"></iframe>

- Reading the bar. Segments on the line are what the boss can roll. The hatched part past the end is what the stats asked for and the ruler could not give.

## Pushed over the edge

- Shield Block. For a few seconds block gets +75 on the sheet. Four rows now ask for far more than the ruler has. Ordinary damage goes first, then the interval above it, crush, then crit.
- The threshold in words. Crush is gone the moment the intervals above it, your four plus the boss's crit, cover the whole ruler. On the sheet that is 102.4 minus the boss's crit chance.
- Where 102.4 comes from. Four rows, 0.6 each, 2.4 in total. It is 100 on the ruler dressed up in sheet units. Nothing magic.
- Why the guides never mention crit. They assume defence has already removed it (440 against a +3 boss). Then crit is 0, and the sheet has to read exactly 102.4. A tank below the defence cap is uncrushable sooner, and crittable.
- "A hair short" and "over the edge" on the ruler. 101.9 leaves crush at 0.5% of swings; one point of dodge and it is zero. A threshold, not a slope. This is the odd thing from the top of the post, explained.
- Fine print. The order crit-above-crush is the wiki and Elitist Jerks model and has not been checked against Classic logs. It only matters for a tank between 96.8 and 102.4 on the sheet. Numbers box.

## What the rule does to the value of a stat

- Marginal product of a point of avoidance under priority: it removes one point of whatever segment is at the bottom of the line. Hit (worth 1.0 of a hit), then crush (1.5), then crit (2.0), then nothing (0).
- Returns are non-monotone: rising in steps as you climb (convex region), then a collapse to zero past the last threshold. Not diminishing returns, the opposite, until the cliff.
- Caps are kinks: the marginal value jumps at each threshold and the last jump is to zero. This is why "cap" is the right word and why 102.4 is not a soft target.
- The sting in the order: near the cap, defence removes a point of crit and lets a point of crush back in. Worked case from the ruler: four rows at 90 on the line, defence 300, crit 5.6, crush 4.4. Raise defence to 440: crit 0, crush 10. Damage per 100 swings 17.8 → 15.0. Naive return 11.2, actual 2.8. Same gear, a point of dodge is worth 1.5.
- Under pro rata none of this happens: marginal values are smooth, small and never collapse. The whole kink structure is an artefact of the priority rule.
- Takeaway for a tank: a stat's worth depends on where the line currently ends, not on the stat. The sheet cannot tell you; the rule can.

<details><summary>The maths</summary>

- Notation: claims $p_k \ge 0$ in table order $k = 1,\dots,7$; the line has length 100; allocation $x_k$ is the probability actually paid.
- Construction A: $x_k = p_k \prod_{j<k}(1 - p_j/100)$.
- Construction B with priority: $x_k = \min\!\big(p_k,\; 100 - \sum_{j<k} x_j\big)$.
- Construction B with pro rata: $x_k = p_k \cdot \min(1, 100/S)$, $S = \sum_k p_k$.

> **Proposition 1 (threshold).** Under priority, crush is paid zero iff $\sum_{\text{miss,dodge,parry,block}} p_k + p_{\text{crit}} \ge 100$. Against a boss $L$ levels up with per-level penalty $\delta$ per row, in sheet units: $\text{sheet sum} \ge 100 + 4L\delta - p_{\text{crit}}$. With $L=3$, $\delta = 0.2$, crit removed: 102.4.

> **Proposition 2 (marginal value).** Expected damage per swing $D = \sum_k w_k x_k/100$. Under priority, $\partial D/\partial p_i = -w_{m}$ for any avoidance row $i$, where $m$ is the lowest row with $x_m > 0$ below $i$; zero if none. Piecewise constant in the sum, non-decreasing in it, then zero.

> **Proposition 3 (crit vs crush).** While crush is partially paid, reducing $p_{\text{crit}}$ by one point changes $D$ by $-(w_{\text{crit}} - w_{\text{crush}})$, not $-w_{\text{crit}}$.

- Proof sketches: each is a direct computation from the allocation formula. Pro rata comparison: $\partial D/\partial p_i$ is a smooth rational function of $S$, never zero.

</details>

> **Numbers as of patch X.Y.**
> - Base miss 5%, base crit 5% vs an equal-level attacker. *(confirmed / inferred?)*
> - Per-level penalty 0.2 per row per level, i.e. 0.04 per skill point and 5 skill per level. *(inferred)*
> - Crushing chance $2 \times (\text{skill} - \text{defence}) - 15$, floored at 0: 15% at +3, none at equal level. *(inferred)*
> - Defence removes 0.04 crit per point above 300; uncrittable vs +3 at 440. *(inferred)*
> - Damage weights: hit 1, crit 2, crush 1.5, blocked hit 0.9 (illustrative; depends on block value vs boss damage). *(inferred)*
> - Shield Block +75% block chance. *(confirmed for 1.12?)*
> - Table order miss, dodge, parry, block, crit, crush, hit with hit cut first: wiki/EJ model. *(inferred; a few thousand logged boss swings on a tank between 96.8 and 102.4 would settle whether crit fills the table.)*

<details><summary>For economists</summary>

- The over-full table is a claims problem: an estate of 100 and claimants with fixed claims $p_k$; the game's job is an allocation rule.
- Priority with a fixed order is the absolute-priority rule; pro rata is the proportional rule; sequential coins are a third rule with no standard name.
- The kinks are complementary slackness: the shadow price of a point of avoidance is the damage weight of the marginal claimant, and it is zero once no claimant is constrained.
- Returns to a stat are locally convex and then collapse. Any model that treats stats as having a constant marginal product, or a diminishing one, gets tank gearing qualitatively wrong near thresholds.
- One pointer to the claims-problems literature (Thomson's surveys); nothing more.

</details>

## Revisions

- v1.0 (15 October 2026): first version.
