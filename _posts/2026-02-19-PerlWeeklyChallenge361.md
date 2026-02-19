---
layout: post
title:  "Perl Weekly Challenge 361: numbers"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 361: numbers

This post presents my solutions to the [Perl Weekly Challenge 361](https://perlweeklychallenge.org/blog/perl-weekly-challenge-361/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 361 - Task 1 - Raku](#task1)
- [PWC 361 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 361 - Task 1 - Raku Implementation

The first task was about finding the Zeckendorf sequence for a given integer.


<br/>
<br/>
```raku
sub MAIN( Int $number is copy
	  where { 0 < $number <= 100 } ) {

    my @fib = 0, 1, 1;
    my @zeckendorf;

    while ( @fib.elems < $number ) {
		@fib.push: @fib[ * - 1 ] + @fib[ * - 2 ];
    }

    for @fib.reverse {
		if ( $_ <= $number ) {
		    @zeckendorf.push: $_;
		    $number -= $_;
		}

        last if $number <= 0;
    }

    @zeckendorf.join( ', ').say;
}

```
<br/>
<br/>


The implementation is quite straighforward: `@fib` is an array that stores the Fibonacci's series up to a given number of elements.
Then I iterate on the reverse Fibonacci's series, and keep all the numbers that build up to the given `$number`, and last I print out the found array.
This solution is clearly not optimized.

<a name="task2"></a>
## PWC 361 - Task 2 - Raku Implementation

Given a matrix of integers, representing booleans that provide information if the current row knows the given columns, find out which *knows everybody*.

<br/>
<br/>
```raku
sub MAIN() {
    my @party =
            [0, 0, 0, 0, 1, 0],  # 0 knows 4
            [0, 0, 0, 0, 1, 0],  # 1 knows 4
            [0, 0, 0, 0, 1, 0],  # 2 knows 4
            [0, 0, 0, 0, 1, 0],  # 3 knows 4
            [0, 0, 0, 0, 0, 0],  # 4 knows NOBODY
            [0, 0, 0, 0, 1, 0],  # 5 knows 4
    ;

    my %known;
    my $required = @party.elems;

    for 0 ..^ @party.elems -> $current {
		my @who =  @party[ $current ].grep( * ~~ 1, :k );
		$required-- and next unless ( @who );
		%known{ $_ }++ for ( @who );
    }

    my @result = %known.keys.grep( { %known{ $_ } == $required } );
    @result.say and exit if ( @result );
    '-1'.say;
}

```
<br/>
<br/>

The implementation counts how many people the `$current` one knows, storing for each person's index the amount of people who know him.
In the case someone knows nobody, it means that we need less `$required` people to satisfy the "knows everyone" concept.

Last, I extract from the `%known` hash the keys (hence, the people) who knows all the others, that means have a counting up to `$required`.
