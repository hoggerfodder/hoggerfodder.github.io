---
title: "The uncrushable cap"
slug: "the-uncrushable-cap"
date: 2026-10-15
lastmod: 2026-10-15
description: ""
tags: []
math: true
draft: true
---

<!-- Talking points are bullets. Replace each block with prose; delete the bullets you don't use. -->

<!-- Opening: the odd thing. No heading; this is the first thing on the page. -->

- Two oddities, one cause. (1) Avoidance on the sheet adds: 5 miss + 8 dodge + 5 parry + 5 block = 23% of swings don't connect. Four independent coins would give $1 - 0.95 \cdot 0.92 \cdot 0.95 \cdot 0.95 = 21.1\%$. (2) Tanks aim for "102.4%", a number no probability can be.
- The same gear that never sees a crushing blow from a level-60 mob gets crushed by a level-63 boss, and one more point of dodge makes it stop entirely. A threshold, not a slope.
- Both come from a single decision about how one swing is resolved. That decision is the subject.

> **The short version.**
> - A swing ends in exactly one of: miss, dodge, parry, block, crit, crush, hit.
> - Stats can be turned into outcome probabilities two ways: a separate coin per stat, or one roll on a line cut into segments. WoW uses one roll, so stats add.
> - One roll forces a rule for what happens when the segments ask for more than the whole line. Blizzard's rule: fill from the top, cut from the bottom.
> - Caps, 102.4 and the crush threshold are all consequences of that rule, not of the stats.

## Probability

- Sample space of a swing. Outcomes are mutually exclusive and exhaustive; probabilities sum to one. That is the only hard constraint; everything else is design.
- What "8% dodge" can mean. (a) Conditional: given the swing is not a miss, 8% of the rest are dodged. (b) Unconditional: 8% of all swings are dodged. Same number on the sheet, different objects.
- Construction A, sequential coins: $P(\text{outcome } k) = p_k \prod_{j<k}(1-p_j)$. Stats compound. Sum of stats is not the probability of avoiding a hit.
- Construction B, one roll: draw $u \sim U[0,100]$, lay segments of length $p_k$ end to end, read where $u$ lands. Stats add. Sum of stats is the probability of avoiding a hit, as long as the segments fit.
- 23 vs 21.1 is the gap between B and A for these numbers. The gap grows with the stats; at raid-tank levels it is several points per swing.
- The stat is not the probability. The construction rule turns stats into probabilities. Keep this line; later episodes stand on it.

## Tables

- The attack table is construction B written out: a fixed order of segments on a line of length 100, one uniform draw, one lookup.
- Order is irrelevant while the segments fit. It becomes everything once they don't.
- Equal-level mob: no crush segment exists. Level-63 boss: a 15-point crush segment appears and each of the four avoidance segments loses 0.6. (Three levels, 0.2 per level per row; four rows; hence 2.4.)
- The ruler. Presets left to right: dungeon gear vs a level-60 mob, the same gear vs level 63, a raid tank with Shield Block down, Shield Block up a hair short, one point over the edge.

<iframe src="ruler.html" width="100%" height="680" loading="lazy" title="Attack table ruler: sliders for miss, dodge, parry and block, a boss level toggle and three rules for resolving the roll"></iframe>

- Reading the bar: segments on the line are what the boss can roll; the hatched part past the end is what the stats claimed but the table could not pay.

## Implementation

- Why a designer picks one roll: one random number per swing, one table lookup, additive stats, a sheet a player can read, and every point of a stat worth the same amount until it isn't.
- The cost of one roll: stats come from gear independently, nothing stops them summing past 100, so the game must say what happens when the claims exceed the line. That rule is a design decision and it is where the caps live.
- Three candidate rules, all on the ruler's toggle:
  - Priority. Fill from the top, cut from the bottom. A segment can be driven to exactly zero. This is what Blizzard shipped.
  - Pro rata. Shrink every segment by the same factor so they fit. Nothing is ever zero; crush just gets small.
  - Sequential coins. Construction A in disguise. Also never zero.
- Under priority, 102.4 is the sheet reading at which your four rows cover the line against a +3 boss. On the line itself it is 100. Nothing magic about the number; it is 100 plus the level penalty.
- Fine print on the order: hit is cut first, then crush, then crit. So crit also fills the table, and a tank who is not yet uncrittable is uncrushable at 102.4 minus the boss's crit chance. The guides quote 102.4 because they assume defence has already removed crit. (Order is the wiki/EJ model, not log-verified; see the numbers box.)

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
