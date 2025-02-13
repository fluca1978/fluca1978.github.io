---
layout: post
title:  "Perl Weekly Challenge 308: lazyness"
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

# Perl Weekly Challenge 308: lazyness

This post presents my solutions to the [Perl Weekly Challenge 308](https://perlweeklychallenge.org/blog/perl-weekly-challenge-308/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 308 - Task 1 - Raku](#task1)
- [PWC 308 - Task 2 - Raku](#task2)



# Raku Implementations

<a name="task1"></a>
## PWC 308 - Task 1 - Raku Implementation


The first task was about counting how many same words are present in two different given arrays.


<br/>
<br/>
```raku
sub MAIN(  :@left,  :@right ) {

    my @matches;

    for @left -> $left {
		@matches.push: $left if ( @right.grep( * ~~ $left ) );
    }

    @matches.elems.say;
}

```
<br/>
<br/>

It is simple to solve with a `grep` and an iteration.


<a name="task2"></a>
## PWC 308 - Task 2 - Raku Implementation

The second task was about to obtain the array that produced an encoded given array given a seed and knowing that each element has been `xor`ed with the previous one.


<br/>
<br/>
```raku
sub MAIN(  :@arr, :$initial  ) {

    my @result = $initial;
    for 1 .. @arr -> $index {
		@result.push: @result[ * - 1 ] +^ @arr[ $index - 1 ];
    }

    @result.say;
}

```
<br/>
<br/>

The trick is to perform the `xor` operations backwards while producing the original array.
