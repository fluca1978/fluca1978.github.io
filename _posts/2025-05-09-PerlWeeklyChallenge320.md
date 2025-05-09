---
layout: post
title:  "Perl Weekly Challenge 320: Simple and Fast!"
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

# Perl Weekly Challenge 320: Simple and Fast!

This post presents my solutions to the [Perl Weekly Challenge 320](https://perlweeklychallenge.org/blog/perl-weekly-challenge-320/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 320 - Task 1 - Raku](#task1)
- [PWC 320 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 320 - Task 1 - Raku Implementation

The first task was about counting how many positive and negative numbers where given as arguments, outputting the max counting value.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems }) {
    ( @numbers.grep( *.Int <= 0 ).elems, @numbers.grep( *.Int >= 0 ).elems ).max.say;
}

```
<br/>
<br/>

A single line does suffice: produce a list with the counting of the values, and then ask the list to get the `max`.


<a name="task2"></a>
## PWC 320 - Task 2 - Raku Implementation

The second task was about printing the difference between the sum of the given numbers and the sum of the digits of the numbers.

<br/>
<br/>
```raku
sub MAIN( *@numbers where { @numbers.grep( *.Int > 0 ).elems == @numbers.elems } ) {
    my $sum = @numbers.sum;
    my $digit-sum = @numbers.map( *.comb ).sum;
    say abs( $digit-sum - $sum );
}

```
<br/>
<br/>

Again, this is quite straightforward to implement.
