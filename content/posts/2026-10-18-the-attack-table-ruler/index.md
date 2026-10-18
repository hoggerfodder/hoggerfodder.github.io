---
title: "The Attack Table Ruler"
slug: "attack-table-ruler"
date: 2026-10-18
lastmod: 2026-10-18
description: "An interactive attack table for Classic WoW tanks. Set miss, dodge, parry, block and defence, pick the boss level, and see which outcomes still fit on the ruler, how far you are from uncrushable, and what one more point of each stat is worth."
tags: []
math: false
draft: false
---

This is the interactive element from the post [Uncrushable at 102.4%: How a Tank Gets Hit](/posts/the-uncrushable-tank/) on its own. The post explains why the ruler looks the way it does. 

<iframe src="/posts/the-uncrushable-tank/ruler.html" width="100%" height="680" loading="lazy" title="The ruler: sliders for miss, dodge, parry and block, a defence skill box and a boss level toggle"></iframe>

## What it shows

One melee swing against you is decided by a single roll from 0 to 100. The possible outcomes are laid out on that line in a fixed order, miss, dodge, parry, block, crit, crush, ordinary hit, and each takes an interval corresponding to its probability of occurrence. Whatever does not fit before the 100 cannot happen.

The **ruler** lane draws those intervals for the boss level you pick. The **sheet** lane above it draws the four defensive numbers at the values the character sheet shows, which are measured against an equal-level attacker. A raid boss three levels above you takes 0.6 off each of the four and gains a crushing-blow interval of 15. The orange mark on the sheet lane is the sum the four have to reach so that block still touches the 100 after that deduction. With the boss's crit reduced to zero it sits at 102.4.

## The controls

- **Sliders** set miss, dodge, parry and block as the sheet shows them. Miss is not on the Vanilla sheet; it is 5 plus 0.04 per point of defence above 300.
- **Defence** is the skill, 300 base at level 60. Each point adds 0.04 to each of the four and takes 0.04 off the boss's crit, so changing it moves the four sliders.
- **Boss level** switches between a same-level mob, which has no crushing blow, and a raid boss three levels above you.
- **Presets** load the gear states used in the post, from dungeon gear against a same-level mob to a raid paladin with Holy Shield up and a Redoubt proc. They are illustrative, not parses.
- The **table** under the sliders lists each interval's length, the part inside the ruler and the part past the edge. The readout above it says how much of the crush interval still lands and how many sheet points would push it off.
- **Marginal value of one more point**, collapsed under the table, shows what a single point of each stat changes in crush, crit and ordinary-hit chance, and the damage it saves per 100 swings. Damage is counted in ordinary hits, with a crit at 2, a crushing blow at 1.5 and a blocked hit at 0.9.

## Limits

- The constants are Vanilla 1.12. Shield Block and Holy Shield enter as block chance, +75 in Vanilla and +30 for Holy Shield in the WoW Forever beta, and their uptime is left to you. Forever may still change numbers before and after launch and I will try to update it; the Numbers box at the end of the post lists what is known.
- Only the outcome roll is modelled, not the damage roll. Block value enters only through the 0.9 weight.
- The settings live in the address of the ruler itself, not of this page. To share a configuration, open [the ruler on its own](/posts/the-uncrushable-tank/ruler.html), set it up and copy the address.

## Revisions

- v1.0 (18 October 2026): first version.
