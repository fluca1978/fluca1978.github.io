---
layout: post
title:  "Perl Weekly Challenge 304: am I lazy?"
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

# Perl Weekly Challenge 304: am I lazy?

This post presents my solutions to the [Perl Weekly Challenge 304](https://perlweeklychallenge.org/blog/perl-weekly-challenge-304/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 304 - Task 1 - Raku](#task1)
- [PWC 304 - Task 2 - Raku](#task2)

I'm a little lazy (and busy) in this period, hence I'm proposing only Raku solutions for this challenge.

# Raku Implementations

<a name="task1"></a>
## PWC 304 - Task 1 - Raku Implementation

Given an array of bits and a positive number, see if it is possible to flip a `0` into a `1` so that no two `1`s are consecutive and the number of flips is the specified positive number.


<br/>
<br/>
```raku
sub MAIN( Int $n, *@bits is copy where { $n > 0 && @bits.elems == @bits.grep( * ~~ /<[01]>/ ).elems } ) {

    my $changes-to-do = $n;

    for 1 ..^ @bits.elems - 2 -> $index {
		if ( @bits[ $index - 1 ] != 1 && @bits[ $index + 1 ] != 1 ) {
		    @bits[ $index ] = 1;
		    $changes-to-do--;
		}
    }

    'True'.say and exit if ( $changes-to-do <= 0);
    'False'.say;
}

```
<br/>
<br/>

The idea is simple: I iterate over all the elements of the array (excluding the boundaries) and try to flip an element if the preceeding and following ones are not already a `1`. Since I do the change in place, because `@bits` is traited with `is copy`, I'm able to examine also the already flips at every loop.

If the `$changes-to-do` has reached zero (or less), the script was succesful.


<a name="task2"></a>
## PWC 304 - Task 2 - Raku Implementation

Given an array of integers and a positive number, find out the subarray of consecutive elements with the same size as the specified number and that have the maximized average.

<br/>
<br/>
```raku
sub MAIN( Int $n, *@numbers where { 0 < $n <= @numbers.elems } ) {

    my @result = 0;

    for 0 ..^ @numbers.elems - $n -> $index {
		my $current-avg   = ( [+] @numbers[ $index .. $index + $n - 1 ] ) / $n;
		my $previous-avg =  ( [+] @result ) / $n;
		@result = @numbers[ $index .. $index + $n - 1 ] if ( $current-avg > $previous-avg );
    }

    ( ( [+] @result ) / $n ).say;
}

```
<br/>
<br/>

At every iteration over the array I consider a subarray, and compute its average. I compare the average with that of the `@result` array, which initially is filled with a single zero so to give an average of zero too.

In the end, I recompute the average and print it out. Note that for caching reasons, it would have been better to store the average computations into variables.
