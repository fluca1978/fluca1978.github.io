---
layout: post
title:  "Perl Weekly Challenge 231: min, max and regexp!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 231: min, max and regexp!

This post presents my solutions to the [Perl Weekly Challenge 231](https://perlweeklychallenge.org/blog/perl-weekly-challenge-231/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 231 - Task 1 - Raku](#task1)
- [PWC 231 - Task 2 - Raku](#task2)
- [PWC 231 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 231 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 231 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 231 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 231 - Task 1 - Raku Implementation

The first task was about extracting all the numbers on a given array of integers excluding the min and max values.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my @wanted;
    @wanted.push: $_ if ( @nums.min < $_ < @nums.max ) for @nums;
    @wanted.join( ',' ).say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 231 - Task 2 - Raku Implementation

The second task was about counting passengers with an age greater than sixty years, given that each passenger data is coded in way that can be easily handled with a regular expression.

<br/>
<br/>
```raku
sub MAIN( *@passengers ) {
    my $count = 0;
    for @passengers {
	next if $_ !~~ / ^ (\d ** 10 ) (<[MF]>) (\d ** 2 ) (\d ** 2) $ /;
	my ( $phone, $gender, $age, $seat ) = $/.Array;
	$count++ if ( $age >= 60 );
    }

    $count.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 231 - Task 1 - PL/Perl Implementation

This time I decided to implement a `min` and `max` functions by myself, so that I can cache the results and loop over the given array to compare every value with the above two.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc231.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   my $min = sub {
      my ( $array ) = @_;
      my $min = undef;

      for ( $array->@* ) {
      	  $min = $_ if ( not defined( $min ) || $_ < $min );
      }

      return $min;
   };


   my $max = sub {
      my ( $array ) = @_;
      my $max = $array->[ 0 ];
      for ( $array->@* ) {
      	  $max = $_ if ( $_ > $max );
      }

      return $max;
   };


   my ( $min_value, $max_value ) = ( $min->( $nums ), $max->( $nums ) );
   for ( $nums->@* ) {
       return_next( $_ ) if ( $_ > $min_value && $_ < $max_value );
   }

   return undef;


$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 231 - Task 2 - PL/Perl Implementation

Here, it is probably even simpler than the Raku implementation, thanks to capturing of regular expressions.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc231.task2_plperl( text[] )
RETURNS int
AS $CODE$
   my ( $passengers ) = @_;
   my $count = 0;

   for ( $passengers->@* ) {
       if ( / ^ \d{10} [MF] (\d{2}) \d{2} $ /x ) {
          $count++ if ( $1 >= 60 );
       }
   }

   return $count;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 231 - Task 1 - PL/PgSQL Implementation

A single query to extract all the values, skipping the min and max coming out of a inner query each.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc231.task1_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
	SELECT
	v
	FROM unnest( nums ) v
	WHERE v <> ( SELECT max( vv ) FROM unnest( nums ) vv )
	AND   v <> ( SELECT min( vv ) FROM unnest( nums ) vv )
	;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 231 - Task 2 - PL/PgSQL Implementation

Here I took a slightly different approach to solve it within a single query.
The `age` view materializes a set of record, each one made by an array of `text[]` because that's the result of `regexp_matches`.
I know every record will be made by a single cell array, so in the end, it just suffice to count the number of records.



<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc231.task2_plpgsql( passengers text[] )
RETURNS int
AS $CODE$

   WITH ages( a ) as (
     SELECT regexp_matches( v::text, '^\d{10}[MF]([6-9]\d)\d{2}$')::text
     FROM unnest( passengers ) v
   )
   SELECT count(*)
   FROM ages



$CODE$
LANGUAGE sql;

```
<br/>
<br/>

In order to avoid the mess of converting a `text[]` to an `int` and compare the value, I included in the regexp the selector to skip every age that is lower than sixty.
