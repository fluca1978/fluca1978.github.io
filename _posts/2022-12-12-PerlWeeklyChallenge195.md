---
layout: post
title:  "Perl Weekly Challenge 195: Bags to the rescue!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 195: Bags to the rescue!

This post presents my solutions to the [Perl Weekly Challenge 195](https://perlweeklychallenge.org/blog/perl-weekly-challenge-195/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 195 - Task 1 - Raku](#task1)
- [PWC 195 - Task 2 - Raku](#task2)
- [PWC 195 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 195 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 195 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 195 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.


<a name="task1"></a>
## PWC 195 - Task 1 - Raku Implementation

The first task was about to find out how many *special integers* there could be from `1` to a given value. A special integer is an integer made by non-repeating digits.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 }, Bool :$verbose = False ) {
    my @special-integers;
    for 1 .. $n {
	  @special-integers.push: $_ if  $_.comb.Bag.values.max <= 1;
    }

    @special-integers.join( ',' ).say if ( $verbose );
    @special-integers.elems.say;
}

```
<br/>
<br/>

The idea is to *classify* the digits that made up every integer using a `Bag`, an hash that counts the repetitions of the keys. If the `max` value of the Bag has a frequency of `1`, it means that all the digits are appearing no more than one time, so there are no repetition, and so the number `$_` can be added to the array of `@special-integers`.
The remaining is just the printing of the number of values.



<a name="task2"></a>
## PWC 195 - Task 2 - Raku Implementation

Given a list of integers, find the most frequent even, and in case there are more than one, find out the lowest one.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.grep( * ~~ Int ).elems == @list.elems } ) {
    my $bag = @list.grep( * %% 2 ).Bag;
    my $most-frequency = $bag.values.max;
    my @most-frequent-evens;
    for $bag.keys {
   	  next if $bag{ $_ } != $most-frequency;
	  @most-frequent-evens.push: $_;
    }

    @most-frequent-evens.min.say;
}

```
<br/>
<br/>

Again, this is solved by means of a `Bag`, first getting out only the even numbers by means of `grep`.
Then I loop over all the keys of the Bag, that are the numbers, and skip the value of the number if it is not classified as one of the maximum frequent ones. Otherwise, if the value in the Bag is equal to the max frequency, I add it to the `@most-frequent-events` array.
Then I pick the `min` value in the array, and that is the searched for value.


<a name="task1plperl"></a>
## PWC 195 - Task 1 - PL/Perl Implementation

Same implementation of the Raku approach, but with the usage of anonymous subroutines to do the Bag-like stuff.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc195.task1_plperl( int, bool default true )
RETURNS int
AS $CODE$
  my ( $n, $verbose ) = @_;
  my @special_integers;

  my $baggify = sub {
     my ( $n ) = @_;
     my $bag = {};
     for my $digit ( split '', $n ) {
     	 $bag->{ $digit }++;
     }

     return $bag;
  };

  my $has_no_repetitions = sub {
     my ( $bag ) = @_;
     for ( keys $bag->%* ) {
     	 return 0 if $bag->{ $_ } != 1;
     }

     return 1;
  };

  for ( 1 .. $n ) {
     push @special_integers, $_ if ( $has_no_repetitions->( $baggify->( $_ ) ) );
  }

  elog( INFO, "Found: " . join( ',', @special_integers ) ) if ( $ verbose );
  return scalar @special_integers;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `$baggify` routine produces a bag like hash, and the `$has_no_repetitions` consides the computed bag and returns a false value if any of the values within the bag is greater than one.
With these two simple functions, it is now straightforward to push every number into the `@special_integers` array and then return the count of the elements in the array.


<a name="task2plperl"></a>
## PWC 195 - Task 2 - PL/Perl Implementation

This is a different approach than the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc195.task2_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $array ) = @_;
   # extract only evens
   my @evens = grep { $_ % 2 == 0 } $array->@*;
   # classify frequency
   my $bag = {};
   $bag->{ $_ }++ for ( @evens );


   # sort by frequency and value
   my @sorted_bag =
     map { $_->[1] }
     sort { $a->[0] <=> $b->[0] || $a->[1] <=> $b->[1] }
     map { [ $bag->{ $_ }, $_ ] } keys $bag->%*;


    # the first value in the list is the
    # one with the max frequency and the lowest value
    return $sorted_bag[ 0 ];
$CODE$
LANGUAGE plperl;
```
<br/>
<br/>

First of all, I `grep` even numbers out of the incoming array, and then classify using a bag like hash.
Then I do use the Schwartz Transform to sort the bag by means of the frequencies and the values. In fact, in the `sort` step I compare first the frequencies, and in the case they are the same the *or* part compares the values. The result is an array of numbers sorted by frequencie and from the lowest to the biggest.
Therefore, it does suffice to return the first value of such array to complete the task.



<a name="task1plpgsql"></a>
## PWC 195 - Task 1 - PL/PgSQL Implementation

A similar implementation to the Raku one.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc195.task1_plpgsql( n int )
RETURNS int
AS $CODE$
DECLARE
	i int;
	freq int;
	counter int := 0;
BEGIN
	FOR i IN 1 .. n LOOP
	    SELECT count(*)
	    INTO   freq
	    FROM regexp_split_to_table( i::text, '' ) as n(d)
	    GROUP BY d
	    ORDER BY 1 DESC;

	    IF freq = 1 THEN
	       counter := counter + 1;
	    END IF;

	END LOOP;

	RETURN counter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


I use `regexp_split_to_table` to split a number into its digits, and then I count the frequency of every digit. If such frequency is different than one, I skip, otherwise I add it to the counting of the special integers.
The idea is that the `SELECT` counts the frequency, so I order it descending having the max value stored into `freq` and therefore if such value is one it means all the digits appear only one, otherwise there are repetitions.


<a name="task2plpgsql"></a>
## PWC 195 - Task 2 - PL/PgSQL Implementation

Here I use a temporary table to store the values and their repetitions.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc195.task2_plpgsql( list int[] )
RETURNS int
AS $CODE$
DECLARE
	current int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS nums( v int, f int default 1, primary key( v ) );
	TRUNCATE TABLE nums;

	FOREACH current IN ARRAY list LOOP
		INSERT INTO nums AS frequency
		SELECT current, 1
		ON CONFLICT (v)
		DO UPDATE SET f = frequency.f + 1;
	END LOOP;

	SELECT v
	INTO current
	FROM nums
	WHERE v % 2 = 0
	ORDER BY f DESC, v ASC
	LIMIT 1;

	RETURN current;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Note that I use an `UPSERT` to store the frequencies of every value.
Then, a single `SELECT` does suffice to extract all even numbers, sort by their frequency and value. The first item returned is the one the task was searching for.
