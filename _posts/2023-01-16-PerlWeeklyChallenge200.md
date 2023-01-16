---
layout: post
title:  "Perl Weekly Challenge 200: not optimal!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 200: not optimal!

This post presents my solutions to the [Perl Weekly Challenge 200](https://perlweeklychallenge.org/blog/perl-weekly-challenge-200/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 200 - Task 1 - Raku](#task1)
- [PWC 200 - Task 2 - Raku](#task2)
- [PWC 200 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 200 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 200 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 200 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 200 - Task 1 - Raku Implementation

The first task was about finding out all *arithmetic slices* of an array of integers greater of three in size. An arithmetic slice is one where the difference between all the elements of the array is constant.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.elems == @list.grep( * ~~ Int ).elems } ) {
    my @slices;

    for 0 ^..^ @list.elems - 1 -> $center {
	for 1 .. $center {
	    my ( $start, $end ) = $center - $_, $center + $_;
	    $start = 0 if $start < 0;
	    $end = @list.elems - 1 if $end >= @list.elems;

	    my @seeking = @list[ $start .. $end ];
	    my $ok = @list.elems >= 3;
	    for 1 ..^ @seeking.elems {
  		   state $difference = @seeking[ $_ ] - @seeking[ $_ - 1 ];
		   $ok = False and last if @seeking[ $_ ] - @seeking[ $_ - 1 ] != $difference;
	    }

	    @slices.push: @seeking if $ok;
	}
    }

    @slices.join( "\n" ).say;
}

```
<br/>
<br/>

While finding out all the slices of a fixed size is trivial, finding out all possibile slices of difference sizes is much more complex.
I decided to center the position of an array into the `$center` index and to move to the left and to the right for a floating size, identified by the `$start` and `$end` indexes. In the nested loop, I compute once (thanks to the `state` declarator) the differnce that must be true into every element of the array, and traverse the `@seeking` subarray (sliced from the oritinal one) to check for every pair.
<br/>
If the `@slices` subarray is fine, I put into the `@slices` container that then I use to print out the result.



<a name="task2"></a>
## PWC 200 - Task 2 - Raku Implementation

Implement a multi-digit LCD 7 segment display.
This was too much hard for me to come with a decent solution!

<br/>
<br/>
```raku
sub MAIN( Int $value = 200, Bool :$sign = False ) {

    my @sign = [
	[ '          ',
	  '          ',
	  '          ',
	  '   -----  ',
	  '          ',
	  '          ',
	  '          ',
	],
	[ '          ',
	  '          ',
	  '     |    ',
	  '   -----  ',
	  '     |    ',
	  '          ',
	  '          ',
	],
    ];

    my @lcd = [
	[ ' -------- ',
	  '|        |',
	  '|        |',
	  '|        |',
	  '|        |',
	  '|        |',
	  ' -------- ',
	],
	[ '         ',
	  '        |',
	  '        |',
	  '        |',
	  '        |',
	  '        |',
	  '         ',
	],
	[ ' ------ ',
	  '       |',
	  '       |',
	  ' ------ ',
	  '|       ',
	  '|       ',
	  ' ------ ',
	],

	[ ' ------ ',
	  '       |',
	  '       |',
	  ' ------ ',
	  '       |',
	  '       |',
	  ' ------ ',
	],

	[ '|      |',
	  '|      |',
	  '|      |',
	  ' ------ ',
	  '       |',
	  '       |',
	  '        ',
	],
	[ ' ------ ',
	  '|       ',
	  '|       ',
	  ' ------ ',
	  '       |',
	  '       |',
	  '        ',
	],
	[ ' ------ ',
	  '|       ',
	  '|       ',
	  ' ------ ',
	  '|      |',
	  '|      |',
	  ' ------ ',
	],

	[ ' -------',
	  '       |',
	  '       |',
	  '       | ',
	  '       |',
	  '       |',
	  '        ',
	],

	[ ' -------- ',
	  '|        |',
	  '|        |',
	  ' -------  ',
	  '|        |',
	  '|        |',
	  ' -------- ',
	],

	[ ' -------- ',
	  '|        |',
	  '|        |',
	  ' -------  ',
	  '         |',
	  '         |',
	  ' -------- ',
	],


    ];

    my @display;
    @display.push: @sign[ $value > 0 ?? 1 !! 0 ] if ( $value < 0 || $sign );
    @display.push: |@lcd[ $value.comb ];
    ( [Z] |@display ).join( "\n" ).say;
}

```
<br/>
<br/>

I placed every *ASCII* representation of a number into an array, so that the digit `0` is at the first position of the array and so on.
Then I put into the `@display` array (used as a container) the ASCII arrays of every digit I extracted from the input number by means of `comb`.
Then I `zip` (thanks to `[Z]`) the arrays so that the first row of the first digit is concatenated with the first row of the second digit and so on, and then I do print all of them.
<br/>
I also placed a `@sign` array containing an ASCII plus and minus to place in front of the digit if required.



# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 200 - Task 1 - PL/Perl Implementation

The same implemetation explained in the Raku solution.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc200.task1_plperl( int[] )
RETURNS SETOF int[]
AS $CODE$
  my ( $list ) = @_;
  my @slices;

  for my $center ( 1 .. $list->@* - 1 ) {
    for ( 1 .. $center ) {
    my ( $start, $end ) = ( $center - $_, $center + $_ );
        $start = 0 if $start < 0;
        $end = $list->@* - 1 if $end >= $list->@*;

        my @seeking = $list->@[ $start .. $end ];
        my $ok = 1;
	    my $difference = undef;

        for ( 1 .. $#seeking ) {
   	      $difference = $seeking[ $_ ] - $seeking[ $_ - 1 ] if ! defined( $difference );
    	  $ok = 0 and last if $seeking[ $_ ] - $seeking[ $_ - 1 ] != $difference;
        }

        return_next( [@seeking] ) if $ok and scalar( @seeking ) >= 3;
    }
  }

return;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 200 - Task 2 - PL/Perl Implementation

The same implementation of the Raku solution, but with a manual *zip operator*.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc200.task2_plperl( int )
RETURNS text
AS $CODE$

   my ( $value ) = @_;

    my $lcd = [
	[ ' -------- ',
	  '|        |',
	  '|        |',
	  '|        |',
	  '|        |',
	  '|        |',
	  ' -------- ',
	],
	[ '         ',
	  '        |',
	  '        |',
	  '        |',
	  '        |',
	  '        |',
	  '         ',
	],
	[ ' ------ ',
	  '       |',
	  '       |',
	  ' ------ ',
	  '|       ',
	  '|       ',
	  ' ------ ',
	],

	[ ' ------ ',
	  '       |',
	  '       |',
	  ' ------ ',
	  '       |',
	  '       |',
	  ' ------ ',
	],

	[ '|      |',
	  '|      |',
	  '|      |',
	  ' ------ ',
	  '       |',
	  '       |',
	  '        ',
	],
	[ ' ------ ',
	  '|       ',
	  '|       ',
	  ' ------ ',
	  '       |',
	  '       |',
	  '        ',
	],
	[ ' ------ ',
	  '|       ',
	  '|       ',
	  ' ------ ',
	  '|      |',
	  '|      |',
	  ' ------ ',
	],

	[ ' -------',
	  '       |',
	  '       |',
	  '       | ',
	  '       |',
	  '       |',
	  '        ',
	],

	[ ' -------- ',
	  '|        |',
	  '|        |',
	  ' -------  ',
	  '|        |',
	  '|        |',
	  ' -------- ',
	],

	[ ' -------- ',
	  '|        |',
	  '|        |',
	  ' -------  ',
	  '         |',
	  '         |',
	  ' -------- ',
	],


    ];


    my $display;

      for my $row ( 0 .. 6 ) {
      	  for ( split '', $value ) {
	      $display .= ' ' . $lcd->[ $_ ]->[ $row ];
	  }

	  $display .= "\n";
      }

      $display .= "\n";

      return $display;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I thougth to use `List::MoreUtils` that provides a `zip` function, but I was unable to instrument it to use a single array as input, and therefore I implemented my own *zip-like* approach.

# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 200 - Task 1 - PL/PgSQL Implementation

A simpler approach, that extracts only fixed 3-sized slices:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc200.task1_plpgsql( list int[] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
BEGIN

	FOR i IN 2 .. array_length( list, 1 ) - 1 LOOP
		IF list[ i + 1 ] - list [ i ] = list[ i ] - list[ i - 1 ] THEN
		   RETURN NEXT array[ list[ i - 1 ], list[ i ], list[ i + 1 ] ];
		END IF;
	END LOOP;

RETURN;

END
$CODE$
LANGUAGE plpgsql;
```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 200 - Task 2 - PL/PgSQL Implementation

I don't believe there is a simple way to implement this in PostgreSQL, so I decided to wrap a Perl implementation.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc200.task2_plpgsql( v int )
RETURNS text
AS $CODE$
   SELECT pwc200.task2_plperl( v );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
