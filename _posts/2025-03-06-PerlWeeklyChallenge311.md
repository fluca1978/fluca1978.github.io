---
layout: post
title:  "Perl Weekly Challenge 311: two lines"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
- python
- java
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 311: two lines

This post presents my solutions to the [Perl Weekly Challenge 311](https://perlweeklychallenge.org/blog/perl-weekly-challenge-311/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 311 - Task 1 - Raku](#task1)
- [PWC 311 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 311 - Task 1 - Raku Implementation


The first task was about to invert the capitalization of letters in a given string.


<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    $string.comb.map( { $_ ~~ /<[A..Z]>/ ?? $_.lc !! $_.uc } ).join.say;
}

```
<br/>
<br/>


The idea is quite simple: I split the string into an array of chars by means of `comb`, then `map` each char to its opposite in upper or lower case, then `join` together all the chars and print out the result.


<a name="task2"></a>
## PWC 311 - Task 2 - Raku Implementation

The task was about splitting a numeric only string into equally sized buckets, and compute the sum of every bucket producing a new string with the result.

<br/>
<br/>
```raku
sub MAIN( Str $digits, Int $size ) {
    $digits.comb.rotor( $size, :partial ).map( { [+] $_ } ).join.say;
}

```
<br/>
<br/>


Raku provides a method to exactly that: `rotor` that splits and array into equally sized sequential sub-arrays (keeping the last one unbalanced, if `:partial` is specified, so that I can `map` every sub array into its sum by means of the reducing `[+]` operator and last I can `join` and print out the result.
