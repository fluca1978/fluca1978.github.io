---
layout: post
title:  "Perl Weekly Challenge 99: overlapping searches"
author: Luca Ferrari
tags:
- raku
- perl-weekly-challenge
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 99: overlapping searches

One way to let me improve my knowledge about Raku (aka Perl 6) is to implement programs in it.
Unluckily, I don't have any production code to implement in Raku yet (sob!).
So, why not try solving the [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks?
<br/>
<br/>
In the following, the assigned tasks for [Challenge 99](https://perlweeklychallenge.org/blog/perl-weekly-challenge-099/){:target="_blank"}.
<br/>
- [Task 1](#task1)
- [Task 2](#task2)




# My eyes...

My last check revealed that there could be a hope, but I have to wait *one year since the surgery* to let the eye to stabilize. After that period, something could be tried.
<br/>
Better than nothing!

<a name="task1"></a>
## PWC 99 - Task 1
The first task required to write a program that accepts a *shell-like* string pattern, where `*` means any char
and `?` means a single character.
<br/>
I've decided to convert the input pattern into a regular expression, with appropriate anchors, and then let Raku do the heavy work for me:

<br/>
<br/>
```raku
sub MAIN( Str $S, Str $P ) {
    # substitute every ? to match a single char
    # and every * to match a stream of chars
    # and add anchors
    my $regexp = '^' ~ $P.subst( '*', '.*' ).subst( '?', '.' ) ~ '$';
    { say 1; exit; } if $S ~~ / <$regexp> /;
    say 0;
}

```
<br/>

The idea is that a pattern like `a*b?c` is translated into the regular expression `^a.*b.c$`, so that the matchin engine can work.



<a name="task2"></a>
## PWC 99 - Task 2

The second task is more complex to me, and *I was unable to fully complete it*. The task required to search for all unique substrings that can be found into another string by composing single characters.
<br/>
After a lot of work with arrays, I decide to use `:exhausistive` option of regular expression, that finds me almost all the required substring matches, but not the right solution:_


<br/>
<br/>
```raku
sub MAIN( Str $S, Str $T ) {
    my $counter = 0;
    my $regexp = $T.comb.join( '.*' );

    # overlapping search
    given $S {
        for m:exhaustive/ <$regexp> / -> $match {
            $counter++;
        }
    }

    say $counter;
}

```
<br/>
<br/>

I do compile the string placing a `.*` between every character, and asking the matching engine to work for me.
*Again, this is not a completely working solution, but I cannot come up with a better solution right now.
