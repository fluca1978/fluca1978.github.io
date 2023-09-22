---
layout: post
title:  "In Perl Parentheses are Optional (except when they are not!)"
author: Luca Ferrari
tags:
- perl
- raku
- programming
permalink: /:year/:month/:day/:title.html
---
Sooner or later every Perl programmer, even experienced ones, will fall into this.

# In Perl Parentheses are Optional (except when they are not!)

In Perl parentheses are optional in many cases, **except when they are not!**

It doesn't matter how much experience you have, sooner or later you will step into this problem, as I did today, and hence the need to remind to other developers this.

I was writing a very simple and tiny line of code:

<br/>
<br/>
```perl
ok ( 0 == grep { $_->authors == 0 } @publications , "There are no publications with no authors" );
```
<br/>
<br/>

The idea is that I've a list of `publication` objects contained into `@publications`, and each object has a method named `authors` that return a list of author names. The test above should check that there are no publications without at least one author.

**The above test is syntactically correct, but semantically wrong!**

Perl refuses to do what I meant: `Can't locate object method "authors" via package "There are no publications with no authors"`.
The error quickly reminds me about the problem.

I am so used to avoid using parentheses around operators, in this case `grep`, that I forget to use them even when the operator is used inside another function, in this case `ok`. Therefore, **`grep` slurps everything after it, in particular `@publications` (what I meant) and `"There are no ... with no authors"` string (not what I meant)**!

The solution is straightforward: add parentheses around `grep` so that Perl understands where the boundaries are.

<br/>
<br/>
```perl
ok ( 0 == grep( { $_->authors == 0 } @publications ), "There are no publications with no authors" );
```
<br/>
<br/>

And the test now runs as I expect and want.


## What about Raku?

Somehow the above problem is less happening to me in Raku. In fact, in order to drop parentheses, you have to use the **precedence dropper** `:`, that triggers in my mind a kind of *be careful from here on*.
