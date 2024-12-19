---
layout: post
title:  "Perl Weekly Challenge 300: only Raku before Christmas!"
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

# Perl Weekly Challenge 300: only Raku before Christmas!

This post presents my solutions to the [Perl Weekly Challenge 300](https://perlweeklychallenge.org/blog/perl-weekly-challenge-300/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 300 - Task 1 - Raku](#task1)
- [PWC 300 - Task 2 - Raku](#task2)
- [PWC 300 - Task 1 in PostgreSQL PL/Perl](#task1plperl)

# Raku Implementations

<a name="task1"></a>
## PWC 300 - Task 1 - Raku Implementation

The first task was about finding all the possible *beautiful permutations* of a given array of integers.



<br/>
<br/>
```raku
sub MAIN( Int $n, Bool :$verbose  where { $n >= 1 } = False ) {

    my @beautifuls;
    for ( 1 .. $n ).permutations -> $current-permutation {
		my $is-beautiful = True;

		for 0 ..^ $current-permutation.elems {
			    my ( $index, $value ) = $_ + 1, $current-permutation[ $_ ];
			    if ( ! ( $value %% $index ) && ! ( $index %% $value ) ) {
					$is-beautiful = False;
					last;
		    }

		}

        @beautifuls.push: $current-permutation if ( $is-beautiful );
    }

    @beautifuls.elems.say;
    @beautifuls.say if ( $verbose );
}

```
<br/>
<br/>

The idea is quite simple: I start iterating over all the permutations, and for each one I iterate over the single elements of the array.
If the element does not respect the *beautifulness rules* I break the loop and mark the whole thing as not beautiful.

In the end, it does suffice to print out how many beautiful permutation I've found.



<a name="task2"></a>
## PWC 300 - Task 2 - Raku Implementation

The second task was about trying to count the max length of possible re-arrangements of an array of integers where the first element is pre-selected on the basis of the current iteration, and the second element is the dereferencing of the first one, the third is the dereferencing of the second one and so on.

Once a duplicated value is found, the iteration stops. The iteration that produces the longest array wins.



<br/>
<br/>
```raku
sub MAIN( *@ints where { @ints.elems == @ints.grep( * ~~ Int ) } ) {

    my @set;

    for 0 ..^ @ints.elems -> $current-set {
		my @current;

		@current[ 0 ] = @ints[ $current-set ];
		for 0 ..^ @ints.elems {
		    next if $_ == $current-set;

		    my $zero-element = @current[ 0 ];
		    my $current-element = @ints[ $zero-element ];
		    my $current-index = $_ - 1;

		    while ( $current-index > 0 ) {
					$current-element = @ints[ $current-element ];
					$current-index--;
		    }

		    last if @current.grep( * ~~ $current-element );
		    @current[ $_ ] = $current-element if ( $current-element );
		}

		@set.push: @current;
    }

    @set.map( *.elems ).max.say;

}

```
<br/>
<br/>

I iterate over the length of the initial array, selecting the first element.
Then I iterate again over the array of integers, skipping the already selected element, and computing the `$current-element` by means of another loop.
If the ongoing array accumulates a duplicated value, I stop the inner loop.
In the end, I remap all the arrays to their length and extract the max value.
