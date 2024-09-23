---
layout: post
title:  "Perl Weekly Challenge 288: not complete!"
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

# Perl Weekly Challenge 288: not complete!

This post presents my solutions to the [Perl Weekly Challenge 288](https://perlweeklychallenge.org/blog/perl-weekly-challenge-288/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 288 - Task 1 - Raku](#task1)
- [PWC 288 - Task 2 - Raku](#task2)
- [PWC 288 - Task 1 in PostgreSQL PL/Perl](#task1plperl)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints.

This week I'm not inspired, and since I've not a good idea about how to implement the second task, I will drop the implementation at the Raku language.

# Raku Implementations

<a name="task1"></a>
## PWC 288 - Task 1 - Raku Implementation


Given an input number, find out the palindrome nearest to it.


<br/>
<br/>
```raku
sub MAIN( $number ) {

    # handling of special case:
    # one digit only
    if ( $number < 10 && $number >= 1 ) {
		say $number - 1;
		exit;
    }

    my ( $left, $right ) = $number - 1, $number + 1;
    while ( $right.Str.flip.Int != $right ) {
		$right++;
    }

    while ( $left.Str.flip.Int != $left ) {
		$left--;
    }

    # what is the one with the lowest difference?
    say $right - $number < $number - $left ?? $right !! $left;
}

```
<br/>
<br/>

The idea is simple: proceed in an iterative way.
Given the `$number`, compute the greater and lower numbers and see if they are palindrome. Having found both, compute the absolute difference and output the one with the smallest difference.


<a name="task2"></a>
## PWC 288 - Task 2 - Raku Implementation

Given a matrix with only two available values, find the dimensions of the boxs for each value.

I don't know how to smartly implement this, and the following solution is not fully correct.

<br/>
<br/>
```raku
sub mark-visited( @positions, $r, $c ) {
    @positions.push: [ $r, $c ] if ( ! @positions.grep( { $_[ 0 ] == $r && $_[ 1 ] == $c } ) );

}


sub MAIN() {
    my $matrix = [
	['x', 'x', 'x', 'x', 'o'],
	['x', 'o', 'o', 'o', 'o'],
	['x', 'o', 'o', 'o', 'o'],
	['x', 'x', 'x', 'o', 'o'],
    ];

    # get all the information about every cell
    # keyed by the cell content
    my %cells = x => Array.new, o => Array.new;

    for 0 ..^ $matrix.elems -> $row {
	  for 0 ..^ $matrix[ $row ].elems -> $col {
	    my $id = $row ~ '-' ~ $col;
	    my $key = $matrix[ $row ][ $col ];
	    mark-visited( %cells{ $key }, $row, $col ) if ( $matrix[ $row ][ $col ] eq $key );
	    # adiacent nodes
	    my ( $nr, $nc ) = $row, $col;
	    $nr = $row + 1;
	    mark-visited( %cells{ $key }, $nr, $nc ) if ( $nr >= 0 && $nr < $matrix.elems && $matrix[ $nr ][ $nc ] eq $key );
	    $nr = $row - 1;
	    mark-visited( %cells{ $key }, $nr, $nc ) if ( $nr >= 0 && $nr < $matrix.elems && $matrix[ $nr ][ $nc ] eq $key );
	    $nr = $row;
	    $nc++;
	    mark-visited( %cells{ $key }, $nr, $nc ) if ( $nc >= 0 && $nc < $matrix[ 0 ].elems && $matrix[ $nr ][ $nc ] eq $key );
	    $nc = $col - 1;
	    mark-visited( %cells{ $key }, $nr, $nc ) if ( $nc >= 0 && $nc < $matrix[ 0 ].elems && $matrix[ $nr ][ $nc ] eq $key );
	  }
    }

    "$_ = { %cells{ $_ }.flat.elems }".say for %cells.keys;
}

```
<br/>
<br/>


The idea is to keep track of already visited cells, and use `mark-visited` as an utility function. Once we grab a cell, add all the contiguos cells that have not been already visited.
In the end, compute the block size.

This solution is not fully correct because it assumes that the blocks are always two, while we can have non-contiguous blocks.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 288 - Task 1 - PL/Perl Implementation

Same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc288.task1_plperl( int )
RETURNS int
AS $CODE$

   my ( $number ) = @_;

   my ( $left, $right ) = ( $number - 1, $number + 1 );

   my $is_palindrome = sub {
      return $_[ 0 ] == join '', reverse split //, $_[ 0 ];
   };

   while ( ! $is_palindrome->( $left ) )  { $left--; }
   while ( ! $is_palindrome->( $right ) ) { $right++; }

   return $number - $left < $right - $number
          ? $left
	  : $right;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

