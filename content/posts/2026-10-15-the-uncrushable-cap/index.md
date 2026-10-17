---
title: "Uncrushable at 102.4%: How a Tank Gets Hit"
slug: "the-uncrushable-tank"
date: 2026-10-17
lastmod: 2026-10-17
description: "Why can a tank's hit probabilities add up to 102.4%? And why is a point of dodge worth more to one tank than to another? One design decision explains both, as well as a few of Classic WoW's stranger quirks that will still matter in Forever."
tags: []
math: true
draft: false
---

Have you heard about a tank in WoW being uncrushable or uncrittable? Maybe you saw some magic 102.4% floating around or some rather cryptic statements about probabilities that don't seem to add up (literally). I certainly have been left puzzled by the character sheet many times and, if anything, knowing a little about probability makes it worse. So this is my attempt to clear up some of this confusion, mostly for myself, hopefully for you too.


> **The short version.**
> - There are seven outcomes of a melee swing but not all of them can always occur.
> - WoW turns your stats into probabilities with one roll on a ruler from 0 to 100, cut into intervals in a fixed order. Stats modify the length of these intervals in a way so that tooltips are exact against an equal-level attacker.
> - Things get interesting when stats increase some intervals so much that another one disappears off the ruler. Now the value of each additional stat changes but this value can increase or drop to 0.
> - A raid boss adds a crushing-blow interval and shaves 0.6 off each of your four defensive outcomes. Cover the ruler and crush is gone. That is the magic 102.4 on the sheet.

## How attacks work

Imagine a game where each attack has a perfectly predictable (deterministic) outcome. You could then tell before you engage whether you or your opponent is going to die. Sounds silly? Chess actually works that way, as a capture always succeeds. Games like Risk and D&D, on the other hand, add an element of chance for some extra spice. You cannot know for sure how each encounter will end and this places us in the realm of probabilities.

Every attack in WoW leads to one outcome chosen from a set of possibilities. For a regular melee attack on a tank, these possibilities usually are:

1. **Miss:** the swing fails, no damage
2. **Dodge:** you get out of the way, no damage
3. **Parry:** you deflect a hit, no damage
4. **Block:** absorbed by shield, reduced damage
5. **Crit:** a serious strike, double damage
6. **Crush:** a bad blow not as bad as crit, 150% damage, only from mobs three or more levels above you, raid bosses included
7. **Ordinary:** normal damage

Randomness enters twice. The ordinary damage is randomly drawn from a range of possibilities and then, independently from that, one of the above outcomes is randomly picked. This leads to no damage, reduced damage, increased damage, or simply the ordinary damage as drawn. The damage draw is simple with a single number picked at random. The other one is where it gets confusing, and that is what I focus on here.

## Probabilities

The seven outcomes above are the sample space of a swing. Each possibility, say a dodge, is an exclusive outcome, meaning there can be only one, and the set is exhaustive, meaning it must be exactly one of those. Each of them can then be assigned a number reflecting its likelihood, and if they are positive and sum to one, they are probabilities. That they sum to one is the only rule the game is bound by. Everything else, including how a number on the character sheet becomes a probability, is a choice Blizzard made.

Of course, we could just uniformly pick an outcome and be done with it, like rolling a seven-sided die, but then there would be no role for gear, buffs, and talents to influence any of this. Instead, each of them is assigned a probability that reflects the stats from the character sheet. Where it gets interesting is how modifiers, like +1% dodge chance, actually affect each probability. Why can we not just add 1% to dodge? The numbers still need to add up to one, so some other probabilities need to give. Which numbers and by how much are the questions that make WoW feel like WoW.

To appreciate how and why things are actually implemented, let's first look at some options *not* chosen by Blizzard. One way to look at this is as a sequence of outcomes. First, miss or not miss. If the attack does not miss, then dodge or not dodge. And so on, until we reach the ordinary damage, which applies if all others did not. So when you pick up a Mark of Tyranny, the trinket from a quest in Upper Blackrock Spire, which says "Increases your chance to dodge an attack by 1%", a natural way to reflect that is to add one percentage point to the dodge step. But dodge is only decided after miss, so what we changed is the chance of a dodge given that the swing did not miss. With a 5% miss chance, the trinket's point arrives as $0.95 \times 1 = 0.95$ points of dodge overall, and every outcome further down the sequence pays for it. It does the job, and yet a tooltip that only gives you a conditional probability seems undesirable. The game could calculate the true effect, but then the tooltip of the same item would differ across players depending on their miss probability.

If we want the stats of the tooltip to be accurate, we should drop the conditional perspective. Instead, we can look at the outcome as the result of a single draw. Consider a ruler with numbers from 0-100. Divide that ruler into several intervals whose length reflects the intended probability. With the numbers from before, miss gets 0-5 for its 5% chance, dodge gets 5-13 for its 8%, parry 13-18, block 18-23, and everything else 23-100. The probabilities then automatically add up to 1 and each outcome corresponds to an interval. To get our random outcome, we then draw a number between 0 and 100 (and unlike a loot roll this doesn't have to be an integer), and the interval the number lands in is the final outcome.

So to reflect an increased dodge chance, we need to lengthen the interval that corresponds to dodge by 1. Since our ruler has a fixed length of 100 numbers, however, we need to shrink some intervals, i.e., we need to normalise everything so we still get probabilities that add up to 1. With dodge at 5-13, we could simply add 1 to each cut-off from 13 onwards. We then get a total length of 101. There are different ways to renormalise this so that we get back to proper probabilities, and they have different properties. Shrinking every interval by the same factor turns the trinket's point into 0.91, which is worse than the conditional / sequential approach. Shrinking only the intervals the trinket did not touch keeps the tooltip exact, but then something even more absurd happens, namely the result depends on which item you put on first, i.e., it would matter in what order you equip your gear. The box below works through both with our numbers.

<details><summary>Deep dive: the three rules Blizzard did not pick</summary>

This box works through alternative rules with the numbers from the text, 5% miss, 8% dodge, 5% parry and 5% block, a Mark of Tyranny for +1% dodge and a Breastplate of Might for +3% block. This is not needed to follow the rest of the post but I find these considerations interesting, so if you want to see exactly where 0.95, 0.91 and the order dependence come from, this is for you.

**A sequence of outcomes.** Miss or not miss; if not, dodge or not dodge; and so on down to ordinary damage. With the trinket's +1% dodge, a natural way to reflect that is to add one percentage point to the dodge outcome (8% becomes 9%), leaving the no-dodge outcome a 91% probability. Notice, however, how dodge is only determined after miss. This means the actual probability of getting a dodge outcome would not increase by 1 percentage point since this only applies IF the opponent does not miss. What we actually modified this way is the conditional probability, the chance of a dodge conditional on not missing. So if not missing has a chance of 95%, then this adjustment only increases the dodge chance overall by $0.95 \times 1 = 0.95$ percentage points. Not missing remains unaffected, while all outcomes further down the order (parry, block, and so on) have a lower probability. It does the job and yet a tooltip that only gives you the conditional probability seems undesirable. The game could, of course, calculate the true effect given the current miss probability, but then the tooltip of the same item would differ across players.
With the sheet reading 5% miss, 8% dodge, 5% parry and 5% block, the sequence turns those into the following shares of all swings, before and after the trinket:

| Outcome | Coin | Before | After +1% dodge | Change |
| --- | --- | --- | --- | --- |
| Miss | 5% | 5.00 | 5.00 | 0 |
| Dodge | 8% → 9% | 7.60 | 8.55 | +0.95 |
| Parry | 5% | 4.37 | 4.32 | −0.05 |
| Block | 5% | 4.15 | 4.11 | −0.04 |
| Everything else | remainder | 78.88 | 78.02 | −0.86 |

The trinket's one point of dodge arrives as 0.95, and every outcome after dodge in the sequence pays for it, in proportion to its size.

**Shrink every interval.** With dodge lengthened to 9 the cut-offs run to 101. We can then multiply each cut-off by 100/101 to make sure the total is right. This rescales each interval proportional to their length (i.e., longer intervals are shortened by more in absolute terms). Dodge asked for 9 of 101 and gets $9 \times 100/101 = 8.91$, so the trinket delivered 0.91 of its point, and every other interval gave up about 1% of itself. That is even less than the 0.95 the sequence delivered. This, however, has the same problem as before - the tooltip does not reflect final outcomes.

**Shrink only the untouched intervals.** You might now object and suggest we should just rescale all the intervals that the trinket hasn't touched. More specifically, we shrink each untouched interval by the same factor, so that together they fit into whatever the trinket left us with. Dodge keeps its full 9. The other intervals add up to 92 and have to fit into 91, so each is multiplied by 91/92. Miss, for instance, goes from 5 to $5 \times 91/92 = 4.95$, and the ordinary-damage interval from 77 to 76.16. Now the tooltip is exact and the trinket delivered its full point.

This sounds great. There is a subtle problem, though. Say you also put on the Breastplate of Might, the tier 1 warrior chest, which says "Increases your chance to block attacks with a shield by 3%" (it also carries some defence, which we ignore here). We can then repeat the procedure so that item also keeps its full point and the untouched intervals shrink to make room. It turns out that the order in which we equipped the items, and thus computed the probabilities, now matters:

| Order | Dodge | Parry | Miss | Block | Everything else |
| --- | --- | --- | --- | --- | --- |
| Trinket first, breastplate second | 8.72 | 4.79 | 4.79 | 7.95 | 73.76 |
| Breastplate first, trinket second | 8.75 | 4.79 | 4.79 | 7.91 | 73.76 |

The difference is small here and yet this is a serious problem. Whatever the sheet shows isn't a function of what you are wearing but of the sequence in which you equipped it. It's easy to see why WoW's designers didn't go down that path.

</details>

Blizzard opted for a rather simple method. When you increase the length of one interval, you cut that same length from the last interval that goes up to 100. So in the case of our example, the ordinary damage one. In other words, you just renormalise a single interval, the one highest up the ruler. All others remain unchanged. Unless we completely push that last interval off the edge, in which case we move to the next outcome on the list.

With our numbers, the trinket turns dodge from 5-13 into 5-14. Every cut-off after it moves up by one, so parry is now 14-19, block 19-24, and the ordinary damage interval runs from 24 to 100 instead of 23. Dodge got its full point, miss, parry and block are untouched, and ordinary damage gets cut. If you add the breastplate, block grows by three, from 19-24 to 19-27, and the ordinary damage interval shrinks further to 27-100. Put the breastplate on first and the trinket second and you land in exactly the same place:

| Outcome | Sheet | Interval after both items | Share of swings |
| --- | --- | --- | --- |
| Miss | 5% | 0-5 | 5 |
| Dodge | 8% + 1% | 5-14 | 9 |
| Parry | 5% | 14-19 | 5 |
| Block | 5% + 3% | 19-27 | 8 |
| Everything else | remainder | 27-100 | 73 |

All four rules on one ruler. Any way of turning the sheet into probabilities is a set of intervals on the same 0-100 line, the sequential evaluation included, so they can be drawn side by side. Tick the items and swap the order:

<iframe src="rules.html" width="100%" height="400" loading="lazy" title="Four rules for fitting a point of dodge onto the ruler: sequential coins, scale everything, keep the point and shrink the rest, last interval pays"></iframe>

With the choices Blizzard made, every tooltip is exact, the sheet adds up, and the order you clicked things in is irrelevant. This seems simple and elegant. Are we done here? There are some subtle complications.

### Mind the kink

How we deal with the probabilities has far more serious consequences than the accuracy of the tooltip. It actually determines the value of each stat. Suppose we increase dodge by 1% and shrink the ordinary damage interval by 1. Then if $D$ is the average (ordinary) damage, this saves one ordinary hit per 100 swings, that is $D$ per 100 swings or $0.01 \times D$ per swing. Per 100 swings is the easier unit, so that is what the rest of this post uses. If instead we shrink each interval proportionally, every interval is multiplied by $100/101$, the ones that benefit us included. Ordinary damage goes from 77 to 76.24 and block from 5 to 4.95, and since a blocked hit still does most of $D$, the saving is about $0.81 \times D$ per 100 swings. The same point of dodge buys four fifths of what it bought under the simple rule. 
Worse, the more avoidance you already have, the smaller the intervals that hurt you, and the less each further point is worth. Each stat has diminishing returns. Under the simple rule a point of dodge is worth $D$ per 100 swings whether it is your first or your fortieth, right up until the ordinary damage interval is gone. Now things get really interesting.

If the ordinary damage interval is pushed over the edge, you no longer take any ordinary damage, it is now a probability 0 outcome. From that point on, we shorten the next interval when you increase your defence stats. Against an enemy of similar level, this is the crit interval. Since a crit multiplies the ordinary damage by 2, shortening that interval has a much greater effect. Added dodge, parry, or miss is now worth twice as much as before. The marginal returns (the returns of each added point) have increased. The more defence you have, the more it is worth (for a while, anyway) but the increase in value is not gradual but jumps.

Block is the odd one out, because a block does not avoid the hit, it only takes a bite out of it. Its marginal value, however, follows the same pattern. If your block value is $B$, a blocked hit still does $D - B$. Say a boss hits for $D = 1000$ and your shield blocks $B = 100$. While ordinary damage is the highest interval, a point of block turns one ordinary hit per 100 swings into a blocked one, saving $D - (D - B) = B$, so 100 damage per 100 swings, while a point of dodge saves the full 1000. Block looks nearly worthless. Once crit is the highest interval, a point of block turns a crit into a blocked hit and saves $2D - (D - B) = D + B$, so 1100, while dodge saves 2000. Block went from a tenth of dodge to more than half of it, by crossing the same edge.
This again continues until crit disappears off the edge. Now the block interval is shortened if we increase miss, dodge, or parry. But since a blocked hit does less damage than a crit or an ordinary hit, the value of each stat decreases at the margin. With the same numbers, a point of dodge now removes one blocked hit per 100 swings and saves $D - B = 900$, down from 2000 a moment ago and below the 1000 it was worth at the start. In fact, if we now increase our block chance, this has no effect at all. The block interval grows by a point at one end and is cut by a point at the other, and nothing on the ruler moves. The value of additional defensive stats first increases and then collapses and each of these changes occurs in discrete jumps whenever a new interval reaches the 100 on the ruler.

Have you ever wondered why gear advice is never simple? This is the reason. The worth of a point of dodge is not a property of dodge. It depends on which interval is highest on the ruler, and that depends on everything else you are wearing, on your buffs, and on who is hitting you. The same trinket is worth 1000 per 100 swings in one build, 2000 in another and 900 in a third, and a point of block can be worth almost nothing or more than half a dodge. A stat weight is a snapshot of one build against one enemy, and it stops being true the moment an interval runs out. BiS might not be BiS for you!


### Cutting from the middle

So far every change came from extending defence intervals and thus cutting the length of intervals from the top. Gear, of course, can also shorten an interval in the middle. Suppose you can reduce the boss's crit chance by one. This then moves the upper crit cut-off down by one. Every cut-off above it slides down with it and the highest interval on the ruler gains the point. What that point is worth again depends on which interval is highest. While ordinary damage can still occur, a 1% reduction in crit becomes a 1% increase in ordinary hit and the reduction saves $2D - D = D$ per 100 swings. A point of crit reduction and a point of dodge increase are worth exactly the same here.

The "cutting from the middle" can lead to some odd situations, though. Suppose your stats are such that the ordinary damage interval is already completely pushed past 100, so you cannot take ordinary damage, and the crit interval also reaches above 100 but still has some part inside the ruler, i.e., straddles the 100. 
Then you can still receive crit damage but any direct crit reduction has no effect. This only moves the upper cut-off down, but it is beyond 100 and therefore irrelevant. The value of the direct crit reduction is 0 before the crit chance is 0. TBC's resilience is a pure crit reducer, so it would behave exactly like this. Whether WoW Forever will add gear of that kind, who knows. (NB: You might ask how the crit interval can reach beyond 100. This is entirely hypothetical but necessary for computing stats when any value decreases. We take the crit chance given your stats if dodge, parry etc. were so low that ordinary damage would still be on the table. If that interval has a length of 20, but given your defensive stats the lower cut-off of crit is 90, then the upper cut-off is 110, with the 100-110 part of the interval ignored.)

In Vanilla, however, nothing exposes this as no gear reduces just crit. The only crit reducer is defence skill, and each point of defence also adds 0.04 to miss, dodge, parry and block. So a point of defence lengthens all four intervals below crit by a total of 0.16, pushing the crit interval further up, thus reducing it in length by 0.16. If crit was fully inside, the same point of defence skill would directly reduce it by 0.04 and the increased intervals of miss, dodge, parry, and block would only shorten the ordinary damage interval. This is the kink again. The defence point has an increasing marginal value, with the change occurring exactly where the ordinary damage disappears off the ruler.

Suppose $D = 1000$ and $B = 100$ and we increase defence by 1 point. The 0.12 of miss, dodge and parry removes that much of the highest interval outright. The 0.04 of block turns the same length of the highest interval into a blocked hit at $D - B$. The 0.04 off crit turns that much crit into whatever the highest interval is. Damage saved per 100 swings:

| Highest interval | 0.12 of it removed | 0.04 of it blocked | 0.04 crit turned into it | Total | Crit cut if it removed crits outright |
| --- | --- | --- | --- | --- | --- |
| Ordinary damage, $D$ | 120 | 4 | 40 | 164 | 80 |
| Crit, $2D$ (straddling) | 240 | 44 | 0 | 284 | 80 |

The increasing returns show clearly. A point of defence as a whole is worth more near the edge than away from it, 284 against 164, in the very row where its crit reduction part achieves nothing. This means a tank stacking defence for crit immunity is actually mostly buying avoidance and thus only indirectly (but very effectively) reducing crit.


### Uncrushable and the 102.4

Once the damage intervals of crit and ordinary are pushed off the edge, you are effectively capped. Yes, you can still exchange block chance for no-damage outcomes like miss, parry, or dodge, but that is it. The only melee damage you take is reduced by your block value. What happened to the crush outcome, though? Classic WoW has yet another mechanic. If you fight enemies $\ge 3$ levels above you, they can land a crushing blow at $1.5 \times$ the ordinary damage. At exactly three levels above you this interval has a length of 15, and it sits above crit. This depends on your level, not on your gear, so unlike crit, you cannot directly reduce it. As we now know, this is immaterial though. We can simply push it off the edge like ordinary damage and crits. And as crush sits closer to the edge than crit, it would in principle disappear first. In practice, however, a fully raid geared tank already eliminates the crit directly without any need to push it off the ruler. But as the crush interval only appears with bosses 3 levels higher, another effect takes place, namely each of miss, dodge, parry, and block loses 0.6 percentage points (0.04 per skill point, five skill points per level). Those intervals shorten. 

Now if crit is eliminated and block already reaches the 100 against ordinary mobs, you would expect crush to disappear. And yet, because of the shorter intervals of miss, dodge, parry, and block against a +3 boss, the upper interval end of block might wander back into the 100, thus allowing for crush damage. How to make sure this never happens against a raid boss? You need miss, dodge, parry, and block to effectively sum up to 102.4, so that block straddles the 100 (the sheet does not show miss; it is 5 plus 0.04 per point of defence above 300). Now when these shrink by 2.4 points, the upper cut-off of block sits right at 100 and the crush chance remains at 0. There it is, uncrushable at 102.4.

Of course, just gear gets you nowhere near that. A fully raid-geared Vanilla warrior sits maybe somewhere around 50 to 60 with everything on. What carries a warrior over the line is Shield Block, +75 block chance, which puts the sheet near 130. We need to be careful here, though. Shield Block is a charge. In Vanilla it covers one block, two with the Improved Shield Block talent, and with block sitting at 93 on the sheet nearly every swing is a block. The charges are gone after one or two swings, and until the next press, 5 seconds later, you are an ordinary tank again with crush back on the ruler. Against a boss swinging every 2 seconds, one charge covers about two swings in five, two charges about four in five. Uncrushable in Vanilla is a question of the share of swings that land while the charge is up.

This still matters in Forever, of course. Crushing blows are still in and the warrior's Shield Block is still +75%, now with two charges over 7 seconds for everyone, which is the old talent's four swings in five without spending the points. A paladin, however, will struggle more. Holy Shield is currently +30% block for 10 seconds with four charges (raised from 20% in the 1 October notes). Four charges on a 10-second cooldown would cover about the same four swings in five, but only for a paladin who reaches the line while it is up, and with Holy Shield at 30 the sheet needs the other 72.4 from gear. A raid-geared paladin sits around 50 to 60, so that is 12 to 22 points short. Redoubt would help on paper. In Vanilla it gives +30% block after you have been critically hit, which an uncrittable tank never is, so it never fires when it matters. In the Forever beta it is a chance to proc on any damaging melee hit, up to 20% block at full rank, which would cut the gap to 52.4. A blocked hit probably counts, so near the line it could keep itself up, but a chance on a hit is still not a guarantee against the next one. Either way, paladins may not be reliably uncrushable at launch or ever (Forever crushable?).

The ruler below lets you walk the whole story. Presets left to right: dungeon gear against a same-level mob, the same gear against a raid boss, a raid warrior with Shield Block down and then up, a fresh 60 with Shield Block up who is a hair short, then one point of dodge over the edge, and a raid paladin with Holy Shield up, without and with a Redoubt proc. The hatched part past the end is the part of each interval that no longer fits on the ruler.

<iframe src="ruler.html" width="100%" height="680" loading="lazy" title="The ruler: sliders for miss, dodge, parry and block, a defence skill box and a boss level toggle"></iframe>

Try the two fresh-60 presets. "A hair short" means just below 102.4 at 101.9 on the sheet. Any raid boss then still crushes (on average) one swing in two hundred. One more point of dodge and that drops to 0 in 100.
Then click the raid warrior with Shield Block up. It gets you to some impressive 129.6, far beyond the 100. Nevertheless, you can still get crushed every time the charges run out. 
The intervals and thus probabilities change dynamically in a fight. The ruler shows you the part the sheet only implies. Which ruler applies in which setting and at what moment in time you still need to figure out yourself. Compute the expected outcome in each, weigh them by how long/often they occur and you will have your expected damage.

> **Numbers as of Vanilla 1.12 and the WoW Forever beta, build 1.60.1.70291.** The Vanilla values are the ones the community established for 1.12, Blizzard never published the table. The Forever values are from the beta spell data and the development notes up to 16 October 2026 and may still change before launch.
> - Base miss 5%, base crit 5% against an equal-level attacker.
> - Per-level penalty 0.2 per row per level, i.e. 0.04 per skill point and 5 skill points per level.
> - Crushing chance $2 \times (\text{attacker skill} - \text{your base defence}) - 15$, floored at 0, with base defence = level × 5: 15% at +3, none at equal level. Defence from gear does not enter.
> - Defence removes 0.04 crit per point above 300 and adds 0.04 to each of miss, dodge, parry and block; uncrittable against +3 at 440.
> - The roll is one number against a running sum from miss upward, ordinary hit is whatever is left, so a shortened interval hands its length to the highest interval on the ruler.
> - Damage weights: hit 1, crit 2, crush 1.5, blocked hit 0.9 (illustrative; depends on block value against boss damage).
> - Shield Block +75% block chance. Vanilla: one block for 5 s, two over 6 s with Improved Shield Block 3/3. Forever: two charges over 7 s baseline.
> - Holy Shield +30% block, 10 s, four charges. Vanilla rank 1.12; Forever raised it back to 30% from 20% on 1 October 2026.
> - Redoubt. Vanilla: +30% block at 5/5 after being critically hit, 10 s or 5 blocks. Forever: 4/8/12/16/20% by rank on damaging melee hits, cut from 6 to 30% on 1 October 2026.



## Revisions

- v1.0 (17 October 2026): first version.
