---
layout: post
title:  "Perl Weekly Challenge 192: distribute and flip"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 192: distribute and flip

It is sad that, after more than three years of me doing Raku, I still don't have any production code project to work on.
Therefore, in order to keep my coding and Raku-ing (is that a term?) knowdledge, I try to solve every  [Perl Weekly Challenge](https://perlweeklychallenge.org/){:target="_blank"} tasks.
<br/>
<br/>
In the following, the assigned tasks for [Challenge 192](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0192/){:target="_blank"}.

<br/>
- [Task 1](#task1)
- [Task 2](#task2)

<br/>
and for the sake of some Perl 5, let's do some stuff also in PostgreSQL Pl/Perl:

<br/>
- [Task 1 in PostgreSQL Pl/Perl](#task1plperl)
- [Task 2 in PostgreSQL Pl/Perl](#task2plperl)


Last, the solutions in PostgreSQL PL/PgSQL:

<br/>
- [Task 1 in PostgreSQL Pl/PgSQL](#task1plpgsql)
- [Task 2 in PostgreSQL Pl/PgSQL](#task2plpgsql)



<a name="task1"></a>
## PWC 192 - Task 1 - Raku Implementation

Given an integer, print the value of the same integer after having flipped all the bits of the input number.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {
    $n.base(2).comb.map( { $_ == 0 ?? 1 !! 0 } ).join.parse-base(2).say;
}

```
<br/>
<br/>

The idea is simple: given `$n` I convert it into binary by means of `base(2)`, then I split into bits (`comb`) and remap every bit to its opposite. Then I do `join` the result obtaining a binary number, that I convert into base-10 by means of `parse-base(2)` and last, I print the result.


<a name="task2"></a>
## PWC 192 - Task 2 - Raku Implementation

Given a list of integers, try to distribute the units of each number so that all the numbers become equal.
<br/>
Therefore, having the list of integers `@n`, I have to try to balance all the elements so that every number looses a unit, and another one gains an unit.

<br/>
<br/>
```raku
sub MAIN( *@n is copy where { @n.elems == @n.grep( * ~~ Int ).elems }
	, :$verbose = False ) {
    my $elem = @n.sum / @n.elems;
    '-1'.say and exit if ( $elem.Int !~~ $elem );

    my @moves;
    @moves.push: [@n];
    while ( @n.grep( * ~~ $elem ).elems != @n.elems ) {
	for 0 ..^ @n.elems -> $index {
	    if ( @n[ $index ] == @n.max ) {
		for 0 ..^ @n.elems -> $borrow {
		    next if $borrow == $index;
		    next if @n[ $borrow ] >= @n[ $index ];
		    next if @n[ $borrow ] >= $elem;
		    @n[ $borrow ]++;
		    last;
		}

		@n[ $index ]--;
	    }

	}

	@moves.push: [@n];
    }


    @moves.join( "\n" ).say if $verbose;
    @moves.elems.say;
}

```
<br/>
<br/>

The first step is to check if the `sum` of the elements can be obtained also with a set of equally valued numbers, otherwise the program terminates.
<br/>
Then I loop until the number of elements in `@n` equal to the average item `$item` is exactly the number of element in the list, that means that the list is balanced.
Within the loop, I do a nested loop, searching for a good item that is the current `max` of the list, so that such element can loose an unit. Having found such element, pointed by `$index`, I search for a different item that has a lower value, so that I can increment the `$borrow`ing element and lower the `$index`ed max value.
<br/>
The resulting array is then stored into `@moves`, so that I can both count how many steps I've done and, under `verbose` print all the moves.

<a name="task1plperl"></a>
## PWC 192 - Task 1 - PL/Perl Implementation

Similar implementation to the Raku one.

<br/>
<br/>
```perl
CREATE SCHEMA IF NOT EXISTS pwc192;

CREATE OR REPLACE FUNCTION
pwc192.task1_plperl( int)
RETURNS int
AS $CODE$
  my ($n) = @_;


  my @bits = map { $_ == 0 ? 1 : 0 } split( '', sprintf( "%b", $n ) );
  my $binary = join( '', @bits );
  my $flipped = eval( "0b$binary" );
  return $flipped;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The trick here is to use `sprintf` to convert a number into binary, and then `eval` to parse back a `ob` string.


<a name="task2plperl"></a>
## PWC 192 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, using anonymous functions to compute the max value in the list and the sum of the array.

<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION
pwc192.task2_plperl( int[] )
RETURNS int
AS $CODE$
   my @moves;
   my ($array) = @_;

   # utility function to get the
   # max value from the array
   my $find_max = sub {
      my $max = 0;
      for ( @_ ) {
        elog(INFO, "value $_" );
        $max = $_ if $_ > $max;
      }

      return $max;
  };

  # utility function to get the sum of the array
  my $sum_array = sub {
     my $sum = 0;
     $sum += $_ for ( @_ );
     return $sum;
  };

  my $item = $sum_array->( $array->@* ) / scalar( $array->@* );
  return -1 if ( $item != int($item) );

  push @moves, $array;

  while ( scalar( grep( { $_ == $item } $array->@* ) ) != scalar( $array->@* ) ) {
         my $max = $find_max->( $array->@* );
  	for my $index ( 0 .. scalar $array->@* ) {
	    next if $array->[ $index ] != $max;

	    for my $borrow ( 0 .. scalar $array->@* ) {
	    	next if $borrow == $index;
		next if $array->[ $borrow ] >= $array->[ $index ];
		next if $array->[ $borrow ] >= $item;
		$array->[ $borrow ]++;
		last;
	    }

	    $array->[ $index ]--;
	}

	push @moves, $array;
  }

  return scalar @moves;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task1plpgsql"></a>
## PWC 192 - Task 1 - PL/PgSQL Implementation

Following the idea behind the other implementations:

<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION
pwc192.task1_plpgsql( n int )
RETURNS int
AS $CODE$
DECLARE
	bb text;
	b  bit;
BEGIN
	bb := '0'; -- needed for the conversion
	FOREACH b IN ARRAY regexp_split_to_array( n::bit(8)::text, '' )  LOOP
		IF b  THEN
		   bb := bb || 0;
		ELSE
		    bb := bb || 1;
		END IF;
	END LOOP;

	RAISE INFO '%', bb;
	RETURN bb::bit(8)::int;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

However, the above code does not return the exact result as the other implementations. The problem is that we should know in advanced the length of the binary conversion (possible, but I'm lazy), therefore an integer is converted into an eight bit string with left padding zeros. Such left padding zeros are then flipped, producing a different output number.


<a name="task2plpgsql"></a>
## PWC 192 - Task 2 - PL/PgSQL Implementation

Cheating here: let's call the PL/Perl implementation. And let's do this in SQL.

<br/>
<br/>
```raku
CREATE OR REPLACE FUNCTION
pwc192.task2_plpgsql( n int[] )
RETURNS int
AS $CODE$
   SELECT pwc192.task2_plperl( n );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
