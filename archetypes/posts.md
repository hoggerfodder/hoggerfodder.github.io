---
title: "{{ path.Base (path.Dir .File.Path) | replaceRE "^\\d{4}-\\d{2}-\\d{2}-" "" | replaceRE "-" " " | title }}"
slug: "{{ path.Base (path.Dir .File.Path) | replaceRE "^\\d{4}-\\d{2}-\\d{2}-" "" }}"
date: {{ .Date | time.Format "2006-01-02" }}
lastmod: {{ .Date | time.Format "2006-01-02" }}
description: ""
tags: []
math: true
draft: true
---

Opening: the odd thing you noticed, in a paragraph or two. The reader should feel the puzzle before seeing the answer.

> **The short version.** The result in plain language, two or three sentences.

Body: the mechanism, explained to a sharp guildmate who hasn't done the maths. Inline maths like $p = 1 - \tfrac{1}{S}$ renders; display maths uses `$$`.

<details><summary>The maths</summary>

The derivation. Propositions as labelled quotes:

> **Proposition 1.** Statement.

Proof sketch, references.

</details>

> **Numbers as of patch X.Y.** Every patch-dependent constant used above, each marked *confirmed* or *inferred*, with what evidence would settle the inferred ones.

<details><summary>For economists</summary>

The one-paragraph aside that names the formal structure (claims problem, complementary slackness, option value, …).

</details>

## Revisions

- v1.0 ({{ .Date | time.Format "2 January 2006" }}): first version.
