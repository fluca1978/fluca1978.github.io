---
layout: post
title:  "Raku: removing duplicates (squish vs unique)"
author: Luca Ferrari
tags:
- raku
- perl
permalink: /:year/:month/:day/:title.html
---
A new function I discovered in Raku

# Raku: removing duplicates (squish vs unique)

Raku is a very fascinating language with a rich and complete set of built-in functions and operators.

I just recently discovered **`squish`**, a function that aims at removing duplicates in a list or `Supply` (see [the function documentation here](https://docs.raku.org/routine/squish){:target="_blank"} ).
<br/>
Wait, isn't that the role of `unique`? Yes, of course, but `squish` works differently: it removes *adiacent repetitions*.

Allow me to explain with a very simple example:

```raku
[3] > my @a = 1,1,2,3,1,4,4,5,5,4;
[1 1 2 3 1 4 4 5 5 4]

[4] > @a.squish;
(1 2 3 1 4 5 4)

[5] > @a.unique;
(1 2 3 4 5)

```

The output of `squish` is a list where **all equal elements that are repeated one after the other are *squashed* to a single value**, while the output of `unique` is a single value no matter how many times it appear and in which positions of the list.

Therefore, `squish` acts like a `unique` on a smaller context, meaning something like *"I already gave you this element, skip its repetition until it needs a retransmission"*.

Raku is amazing!
