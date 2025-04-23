---
layout: post
title:  "Perl Weekly Challenge 318: Short and to the point!"
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

# Perl Weekly Challenge 318: Short and to the point!

This post presents my solutions to the [Perl Weekly Challenge 318](https://perlweeklychallenge.org/blog/perl-weekly-challenge-318/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 318 - Task 1 - Raku](#task1)
- [PWC 318 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 318 - Task 1 - Raku Implementation

Given a string of lower case letters, find out all the groups of 3 consecutive characters.



<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    $string.comb.unique.map( -> $char { ( $string ~~ / $char ** 3 / // '' ).Str } )
		          .grep( { $_.chars == 3 } )
		          .join( ', ' ).say;
}

```
<br/>
<br/>


This can be solved with a single line: `map` the `unique` set of chars to a regular expression that finds out a consecutive cluster of three chars, then filter out the empty strings and print out.



<a name="task2"></a>
## PWC 318 - Task 2 - Raku Implementation

Given two arrays of integers, find out if reverting a sub-array of one can lead to the other.

<br/>
<br/>
```raku
sub MAIN( :@a, :@b) {

    for 0 ^..^ @a.elems {
		'True'.say and exit if ( @a[ 0 .. $_ ].reverse, @a[ $_ + 1 .. * - 1 ] ).flat ~~ @b;
    }

    'False'.say;
}

```
<br/>
<br/>

The idea is to `revert` a subscript of the `@a` array, joining with the remaining items and *smart matching* against the other array.
