---
layout: post
title:  "Perl Weekly Challenge 344: Lazyness and too much tasks!"
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

# Perl Weekly Challenge 344: Lazyness and too much tasks!

This post presents my solutions to the [Perl Weekly Challenge 344](https://perlweeklychallenge.org/blog/perl-weekly-challenge-344/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 344 - Task 1 - Raku](#task1)
- [PWC 344 - Task 2 - Raku](#task2)


**I'm becoming lazy, or better, I'm so busy that I don't have time to enjoy the PWC!*


# Raku Implementations

<a name="task1"></a>
## PWC 344 - Task 1 - Raku Implementation

The first task was about to *array sum* a given array with a number. The idea is to disassemble the number so that each cell represents a single digit, then sum the arrays together.

<br/>
<br/>
```raku
sub MAIN( Int $x, *@numbers where { @numbers.grep( * ~~ Int ).elems == @numbers.elems } ) {
    my @n = $x.comb;
    @n.unshift: 0  while ( @n.elems < @numbers.elems );

    say @numbers <<+>> @n;
}

```
<br/>
<br/>

This is simple enough in Raku, since the `<<+>>` operator does the trick once the number has been *array*-ized via `comb`.



<a name="task2"></a>
## PWC 344 - Task 2 - Raku Implementation

The second task was to find out if a given list of list can be rearranged into a flat given list, without having to break any sublist.


<br/>
<br/>
```raku
sub MAIN( ) {
    my @source = [2,3], [1], [4];
    my @destination = 1,2,3,4;

    #@source = [9,1], [5,8], [2];
    #@destination = 5, 8, 2, 9, 1;

    # @source = [1], [3];
    # @destination = 1,2,3;

    my $destination-as-string = @destination.sort.join;

    my $found = 0;
    for @source -> $current {
		my $index = Nil;

        for $current.List -> $x {
		    'False'.say and exit if ( ! @destination.grep( * ~~ $x ) );
		    if ( ! $index ) {
				$index = @destination.grep( * ~~ $x, :k ).first;
		    }
		    else {
				'False'.say and exit if ( @destination[ ++$index ] != $x );
		    }

		    $found++;
		}
    }



    'True'.say and exit if ( $found == @destination.elems );
    'False'.say;
}

```
<br/>
<br/>

The idea is to iterate over the list of lists and see if the current element is contained in the flat list, if no the program stops since there is a failure.
Otherwise, I seek for a new value out of the final list and then iterate again to see if the next index contains the next element of the sublist.

