---
layout: post
title:  "Perl Weekly Challenge 354: time to drink and wait for the new year!"
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

# Perl Weekly Challenge 354: time to drink and wait for the new year!

This post presents my solutions to the [Perl Weekly Challenge 354](https://perlweeklychallenge.org/blog/perl-weekly-challenge-354/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 354 - Task 1 - Raku](#task1)
- [PWC 354 - Task 2 - Raku](#task2)

# Raku Implementations

<a name="task1"></a>
## PWC 354 - Task 1 - Raku Implementation


The first task was about finding all the couple of numbers that provide the min difference in a given list.

<br/>
<br/>
```raku
sub MAIN( *@nums is copy
	  where { @nums.elems %% 2 && @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    @nums .= sort;
    my %diffs;
    @nums[ 0 .. * - 2].map( { state $i = 0;
			      %diffs{ @nums[ $i + 1 ] - @nums[ $i ] }.push: [ @nums[ $i + 1 ], @nums[ $i ] ];
			      $i++;
			    } );

    %diffs{ %diffs.keys.map( *.Int ).min }.join( "\n" ).say;
}

```
<br/>
<br/>

I do this:
- `sort` the numbers (in place)
- `map` all the couples into a `%diffs` hash where the key is the value of the difference, and the value is an array where I push all the couples with the same difference.

After having done that, I can extract the `min` key and print out all the couples.



<a name="task2"></a>
## PWC 354 - Task 2 - Raku Implementation

The second task was about shifting a matrix following a few given rules:
- every element moves to its right;
- a line wraps to the next one;
- the last element wraps to the first.

I provide the application with all the given values in `@matrix` and with `$k` as the number of shits.

<br/>
<br/>
```raku
sub MAIN() {
    my @matrix = [1, 2, 3],
                 [4, 5, 6],
                 [7, 8, 9];
    my $k = 1;

    @matrix = [1, 2],
                  [3, 4],
                  [5, 6];
    $k = 1;

    @matrix = [1, 2, 3],
              [4, 5, 6];
    $k = 5;

    my @next;
    while ( $k ) {
		for 0 ..^ @matrix.elems -> $row {
		    if ( $row == 0 ) {
				@next[ $row ].push: @matrix[ * - 1 ][ * - 1 ];  # element at 0, 0
				@next[ $row ].push: |@matrix[ $row ][ 0 .. * - 2 ]; # first row
		    }
		    else {
				@next[ $row ].push: @matrix[ $row - 1 ][ * - 1 ];
				@next[ $row ].push: |@matrix[ $row  ][ 0 .. * - 2 ];
		    }
		}

		$k--;
		if ( $k ) {
		    @matrix = @next;
		    @next = ();
		}
    }


    say @next;
}

```
<br/>
<br/>

The idea is quite simple: iterate on all the `$row`s of the matrix and, in the case of the first row, construct an ad-hoc row, otherwise build it on the basis of the previous one.
Then switch the original matrix with the new one and restart.
