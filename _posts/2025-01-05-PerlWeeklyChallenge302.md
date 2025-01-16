---
layout: post
title:  "Perl Weekly Challenge 302: first challenge of the year"
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

# Perl Weekly Challenge 302: first challenge of the year

This post presents my solutions to the [Perl Weekly Challenge 302](https://perlweeklychallenge.org/blog/perl-weekly-challenge-302/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 302 - Task 1 - Raku](#task1)
- [PWC 302 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 302 - Task 1 - Raku Implementation

Given a set of binary strings and a number `x` of `1`s and `y` of `0`s, find out the largest size of a subset so that there are no more `$x` and `$y` conditions.



<br/>
<br/>
```raku
sub MAIN( Int :$x, Int :$y, *@str ) {

    my %solutions;

    for @str.permutations -> $array {
		my @current-set;

		for $array.Array -> $_ {

		    if ( @current-set.join.grep( * ~~ 0 ).elems <= $y
				&& @current-set.join.grep( * ~~ 1 ).elems <= $x ) {
				@current-set.push: $_;
		    }
		    else {
				last;
		    }

		}

		%solutions{ @current-set.elems }.push: @current-set if ( @current-set );
    }

    %solutions.keys.max.say;
}

```
<br/>
<br/>

The idea is quite simple: I iterate over all possible permutations of the array of binary strings, and build an array of element (strings) named `@current-set`. Every time I add a new string from the current permutation, I ensure that the `$x` and `$y` conditions are met, and once they are not met anymore I add the solution to the `%solutions` hash, keyed by the length of the subset.

Therefore, it does suffice to grab the highest key.



<a name="task2"></a>
## PWC 302 - Task 2 - Raku Implementation


Given an array of integers, find out a minimum start so that the sum of each step is greater than one.


<br/>
<br/>
```raku
sub MAIN() {

    my @nums = -3, 2, -3, 2, 4;
    for 0 .. Inf -> $start {
		my $current = $start;
		my $ok = True;

		for @nums {
		    $current += $_;
		    if ( $current < 1 ) {
				$ok = False;
				last;
		    }
		}

		$start.say and exit if ( $ok );
    }
}

```
<br/>
<br/>

I iterate on every possible number, and fo each value assumed as a start, I iterate over the array to ensure there is no step less than 1.
At the first value that ensures me the condition, I print it and terminate the program.
