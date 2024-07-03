---
layout: post
title:  "Perl Weekly Challenge 276: filtering arrays"
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

# Perl Weekly Challenge 276: filtering arrays

This post presents my solutions to the [Perl Weekly Challenge 276](https://perlweeklychallenge.org/blog/perl-weekly-challenge-276/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 276 - Task 1 - Raku](#task1)
- [PWC 276 - Task 2 - Raku](#task2)
- [PWC 276 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 276 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 276 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 276 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 276 - Task 1 in PL/Java](#task1pljava)
- [PWC 276 - Task 2 in PL/Java](#task2pljava)
- [PWC 276 - Task 1 in Python](#task1python)
- [PWC 276 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 276 - Task 1 - Raku Implementation

The first task was about finding all the couples of a given list of integers so that the sum of every couple is a multiple of `24` (hours).
This can be solved with a one line.

<br/>
<br/>
```raku
sub MAIN( *@hours where { @hours.elems == @hours.grep( * ~~ Int ) } ) {
    @hours.combinations( 2 ).grep( { ( $_[ 0 ] + $_[ 1 ] ) %% 24 } ).say;
}

```
<br/>
<br/>


The `combinations( 2 )` provides me all the possible combinations of two elements, therefore a couple, and then I keep by means of `grep` only thos that have a sum that is a multiple of `24`.

<a name="task2"></a>
## PWC 276 - Task 2 - Raku Implementation

Given an array of integers, tell how many elements have the same max frequency.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( { $_ ~~ Int && $_ > 0 } ).elems } ) {
    my %frequency;
    %frequency{ @nums.grep( * ~~ $_ ).elems }.push: $_ for @nums;
    %frequency{ %frequency.keys.max }.unique.elems.say;
}

```
<br/>
<br/>

First of all, I classify the `%frequency` of every element, then extract the max value of the keys (frequency), pass thru `unique` to exclude duplicates, and count the number of `elems`.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 276 - Task 1 - PL/Perl Implementation

I need to use an external module `Algorithm::Combinatorics`, so the implementation must be done in `plperlu`.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc276.task1_plperl( int[] )
RETURNS SETOF int[]
AS $CODE$
   use Algorithm::Combinatorics qw/ combinations /;

   my ( $hours ) = @_;
   my $iterator = combinations( \ $hours->@*, 2 );
   while( my $c = $iterator->next ) {
   	  return_next( $c ) if ( ( $c->@[ 0 ] + $c->@[ 1 ] ) % 24 == 0 );
   }

   return undef;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

I return every couple of the `combinations` that satisfies the `24` sum multiple.



<a name="task2plperl"></a>
## PWC 276 - Task 2 - PL/Perl Implementation


The approach is the same as in Raku, only more verbose.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc276.task2_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   die "Need only positives" if ( grep { $_ <= 0 } $nums->@* );

   my $frequency = {};
   for my $current ( $nums->@* ) {
       my $count = scalar grep { $_ == $current } $nums->@*;
       push $frequency->{ $count }->@*, $current if ( ! grep { $_ == $current } $frequency->{ $count }->@* );
   }

   my $max_frequency = ( sort( { $b <=> $a } keys $frequency->%* ) )[ 0 ];
   return scalar $frequency->{ $max_frequency }->@*;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

First I build the `$frequency` hash with the frequencies as keys and the list of unique elements as the values.
Then I compute the `$max_frequency` using `sort` and limiting the number of results, and last I compute how many elements appear with such frequency.



# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 276 - Task 1 - PL/PgSQL Implementation

A single query does suffice!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc276.task1_plpgsql( hours int[] )
RETURNS TABLE( l int, r int )
AS $CODE$

   WITH elems AS ( SELECT v::int, row_number() OVER ( ORDER BY v ) AS r
                   FROM unnest( hours ) v
		 )

   SELECT l.v::int, r.v::int
   FROM elems l, elems r
   WHERE mod( ( l.v::int + r.v::int ), 24 ) = 0
   AND   l.r < r. r
   ;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The idea is to produce a materialization of the array elements with a row numbering, so that I can join the materialization against itself and get only those pairs that produce a modulo `24` value assuming the row number of one is greater than the other (to skip self-sums).


<a name="task2plpgsql"></a>
## PWC 276 - Task 2 - PL/PgSQL Implementation


Again, a single query!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc276.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
   WITH freq AS (
   	SELECT count( v ) as frequency, v
   	FROM unnest( nums ) v
   	GROUP BY v
   )
   , max_freq AS ( SELECT frequency FROM freq
                   ORDER BY 1 DESC
		   LIMIT 1
		 )
	SELECT count( f )
	FROM   freq f
	WHERE f.frequency = ( SELECT frequency FROM max_freq )

   ;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

First I materialize the `freq` list of frequency and values. Then, on top this, I build the `max_freq` present in the above list.
Last, I count how many rows I have for the the `max_freq`.

# Java Implementations

<a name="task1pljava"></a>
## PWC 276 - Task 1 - PostgreSQL PL/Java Implementation

A boring nested loop to solve this.
<br/>
<br/>
```java
   public static final String[] task1_pljava( int[] hours ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc276.task1_pljava" );

		List<String> result = new LinkedList<String>();
		for ( int i = 0; i < hours.length - 1; i++ )
		    for ( int j = i + 1; j < hours.length; j++ )
			if ( ( hours[ i ] + hours[ j ] ) % 24 == 0 )
			    result.add( String.format( "%02d + %02d", hours[ i ], hours[ j ] ) );

		return result.toArray( new String[ 0 ] );
    }
```
<br/>
<br/>

I return a `String[]` just to avoid the cluttering of implementing a complex `ResultProvider`.



<a name="task2pljava"></a>
## PWC 276 - Task 2 - PostgreSQL PL/Java Implementation

Using the Stream API this can be solved in a more compact way.

<br/>
<br/>
```java
    public static final int task2_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc276.task2_pljava" );

		final Map<Integer, List<Integer>> frequency = new HashMap<Integer, List<Integer>>();

		Arrays.stream( nums )
		    .forEach( v -> {
			    int freq = (int) Arrays.stream( nums ).filter( k -> k == v ).count();
			    frequency.putIfAbsent( freq, new LinkedList<Integer>() );
			    if ( ! frequency.get( freq ).contains( v ) )
				 frequency.get( freq ).add( v );
			} );

		final int[] max = new int[]{ 0, 0 };
		// find out the max frequency
		frequency.keySet().stream().forEach( k -> {
			if ( k > max[ 0 ] ) {
			    max[ 0 ] = k;
			    max[ 1 ] = frequency.get( k ).size();
			}
		    } );

		return max[ 1 ];
    }
```
<br/>
<br/>

The first iteration with `forEach` is done to map into `frequency` the frequency found and the list of values that have such frequency.
The second `forEach` is done over the keys of the `Map` (i.e., the frequencies) to find the greatest one and store into `max`, along with the length of the values.


# Python Implementations

<a name="task1python"></a>
## PWC 276 - Task 1 - Python Implementation

Thanks to `itertools` this can be solved in one line (or so).
<br/>
<br/>
```python
import sys
from itertools import combinations

# task implementation
# the return value will be printed
def task_1( args ):
    hours = list( map( int, args ) )
    return list( filter( lambda v: ( v[ 0 ] + v[ 1 ] ) % 24 == 0,  list( combinations( hours, 2 ) ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 276 - Task 2 - Python Implementation

The implementation is really similar to the PL/Perl one, except that I keep track of the max frequency while iterating to avoid a filtering on the dictionary keys.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    nums      = list( map( int, args ) )
    frequency = {}
    max_freq  = 0

    for x in nums:
        freq = len( list( filter( lambda v: v == x, nums ) ) )
        if freq > max_freq:
            max_freq = freq

        if not freq in frequency:
            frequency[ freq ] = []

        if not x in frequency[ freq ]:
            frequency[ freq ].append( x )


    return len( frequency[ max_freq ] )

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
