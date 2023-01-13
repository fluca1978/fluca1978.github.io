---
layout: post
title:  "Perl Weekly Challenge 198: First Perl Code of the Year!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 198: First Perl Code of the Year!

This post presents my solutions to the [Perl Weekly Challenge 198](https://perlweeklychallenge.org/blog/perl-weekly-challenge-198/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 198 - Task 1 - Raku](#task1)
- [PWC 198 - Task 2 - Raku](#task2)
- [PWC 198 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 198 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 198 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 198 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.


<a name="task1"></a>
## PWC 198 - Task 1 - Raku Implementation

The first task was about finding, in an unordered integer list, pairs that when ordered have the same distance between the units.

<br/>
<br/>
```raku
sub MAIN( *@list where { @list.elems == @list.grep( * ~~ Int ).elems } ) {
    my @sorted = @list.sort;

    my %pairs;
    for 1 ..^ @sorted.elems - 1 {
		if ( ( @sorted[ $_ ] - @sorted[ $_ - 1 ] ) == ( @sorted[ $_ + 1 ] - @sorted[ $_ ] ) ) {
		    %pairs{ @sorted[ $_ ] - @sorted[ $_ - 1 ] }.push: @sorted[ $_ - 1, $_ ] , @sorted[ $_,  $_ + 1 ];
		}
    }


    %pairs{ %pairs.keys.max }.elems.say;

}

```
<br/>
<br/>

The trick here is to `sort` the input list, and then iterate over every position starting from the second one in order to see if the difference between the current element and the previous one is the same as the difference between the current element and the next one. In such case, both the pairs are stored into the `%pairs` hash, keyed by the value of the difference.
<br/>
Last, I extract the max value of the keys and get the list of elements and its size.

<a name="task2"></a>
## PWC 198 - Task 2 - Raku Implementation

Find out the number of prime numbers less than a given input number.

<br/>
<br/>
```raku
sub MAIN( Int $n where { $n > 0 } ) {
    (2 .. $n).grep( *.is-prime ).elems.say;
}

```
<br/>
<br/>

This can be solved really simply in Raku: I get the range of integers from `2` to `$n` and ask for all the elements that are primes by means of the method `is-prime`, and then I count the number of elements.


<a name="task1plperl"></a>
## PWC 198 - Task 1 - PL/Perl Implementation
A solution cloned from the Raku approach.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc198.task1_plperl( int[] )
RETURNS int
AS $CODE$
  my ( @list ) = sort $_[0]->@*;
  my $pairs = {};
  for ( 1 .. @list - 1 ) {
      if ( ( $list[ $_ ] - $list[ $_ - 1 ] ) == ( $list[ $_ + 1 ] - $list[ $_ ] ) ) {
      	 push $pairs->{ $list[ $_ ] - $list[ $_ - 1 ] }->@*, $list[ $_ ], $list[ $_ - 1 ], $list[ $_ + 1 ], $list[ $_  ];
      }
  }

  my $max = 0;
  for ( keys $pairs->%* ) {
      $max = $_ if $_ > $max;
  }

  return scalar $pairs->{ $max }->@*;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `$pairs` is the hash reference that contains the list of pairs and is keyed by their difference. I need to manually extract the `$max` key and then return the size of the array contained at such key.


<a name="task2plperl"></a>
## PWC 198 - Task 2 - PL/Perl Implementation

More verbose than the Raku solution, since there is no built-in prime checking in Perl and I don't want to use a library.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc198.task2_plperl( int )
RETURNS int
AS $CODE$
   my ( $n ) = @_;

   my $is_prime = sub {
      for ( 2 .. $_[0] - 1 ) {
      	  last if $_ * 2 > $_[0];
      	  return 0 if $_[0] % $_ == 0;
      }

      return 1;
   };

   my $counter = 0;
   $counter += $is_prime->( $_ )  for ( 2 .. $n );

   return $counter;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The `$is_prime` subroutine does a cowly checking of a number and returns `0` if it is not prime, else it returns `1`.
This makes simple to count how many prime numbers are there in a range, since I can sum all the results of calling `$is_prime` within a loop.



<a name="task1plpgsql"></a>
## PWC 198 - Task 1 - PL/PgSQL Implementation

This time I decided to implement the solution with a pure query.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc198.task1_plpgsql( l int[] )
RETURNS int
AS $CODE$

   with counting as (
      select v, v - lag( v, 1, v ) over w as d
      from unnest( l ) v
      window w as (order by v asc )
   )
   , max_counting as (
     select max( d ) from counting
   )
   select count(*)
   from counting
   where d = ( select * from max_counting );


$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The Common Table Expression is made by two parts: `counting` provides a table with a number `v` and its difference from the previous one, by means of its `lag`. The `unnest` transforms the array into a table and the `order by v asc` provides the sorting of the input values.
The second part is `max_counting` that extracts the max difference from the pairs in `counting`.
Last, I count all the pairs that have the max value of difference.


<a name="task2plpgsql"></a>
## PWC 198 - Task 2 - PL/PgSQL Implementation

Similar to the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc198.is_prime( l int )
RETURNS bool
AS $CODE$
DECLARE
	i int;
BEGIN
	FOR i IN  2 .. l - 1 LOOP
	    IF l % i = 0 THEN
	       RETURN FALSE;
	    END IF;
	END LOOP;

	RETURN TRUE;
END
$CODE$
LANGUAGE plpgsql;



CREATE OR REPLACE FUNCTION
pwc198.task2_plpgsql( l int )
RETURNS int
AS $CODE$
DECLARE
	c int := 0;
	i int;
BEGIN
	FOR i IN 2 .. l LOOP
	    IF pwc198.is_prime( i ) THEN
	       c := c + 1;
	    END IF;
	END LOOP;

	RETURN c;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

I use a routine `is_prime` to test if a number is prime, and then loop over all the numbers in the range and add one unit to the counting when a number is prime.
