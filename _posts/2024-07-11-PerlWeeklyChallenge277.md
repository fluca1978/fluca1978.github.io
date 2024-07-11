---
layout: post
title:  "Perl Weekly Challenge 277: arrays and lists"
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

# Perl Weekly Challenge 277: arrays and lists

This post presents my solutions to the [Perl Weekly Challenge 277](https://perlweeklychallenge.org/blog/perl-weekly-challenge-277/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 277 - Task 1 - Raku](#task1)
- [PWC 277 - Task 2 - Raku](#task2)
- [PWC 277 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 277 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 277 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 277 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 277 - Task 1 in PL/Java](#task1pljava)
- [PWC 277 - Task 2 in PL/Java](#task2pljava)
- [PWC 277 - Task 1 in Python](#task1python)
- [PWC 277 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 277 - Task 1 - Raku Implementation

The first task was about finding common words in two list of words, where each word appaears only once in each list.

<br/>
<br/>
```raku
sub MAIN( :@left, :@right ) {
    my %left-bag  = Bag.new( @left );
    my %right-bag = Bag.new( @right );
    my @matches;
    my @single-left  = %left-bag.kv.grep( -> $k, $v { $v == 1 } ).map( *[ 0 ] );
    my @single-right = %right-bag.kv.grep( -> $k, $v { $v == 1 } ).map( *[ 0 ] );
    for @single-left -> $left {
		@matches.push( $left ) if ( @single-right.grep( * ~~ $left ) );
    }

    @matches.elems.say;
}

```
<br/>
<br/>


I first create a `Bag` to count how many times a word appears in every list, then I filter out the single words from each list.
Last I do a loop to find out which words do match, storing them into the `@matches` array and then providing the count of the elements.



<a name="task2"></a>
## PWC 277 - Task 2 - Raku Implementation

Given a list of integers, find out which pairs are *strong*: a strong pair has an absolute value of the difference less than the min value of the pair.


<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    # 	A pair of integers x and y is called strong pair if it satisfies: 0 < |x - y| < min(x, y).
    my @strong;
    for 0 ..^ @nums.elems - 1 -> $i {
		for $i ^..^ @nums.elems -> $j {
	    @strong.push: [ @nums[ $i ], @nums[ $j ] ] if ( 0 < abs( @nums[ $i ] - @nums[ $j ] )  < min( @nums[ $i ], @nums[ $j ] ) );
		}
    }

    @strong.elems.say;
}

```
<br/>
<br/>


I use a nested loop to check the condition.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 277 - Task 1 - PL/Perl Implementation

I keep two lists of words where I store only the single appearing words in every list.

Then, with a loop, I compare the two single-word lists one against the other to find out the matches.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc277.task1_plperl( text[], text[] )
RETURNS int
AS $CODE$

   my ( $left, $right ) = @_;

   my @single_left;
   my @single_right;

   for my $word ( $left->@* ) {
       push @single_left, $word  if ( scalar( grep { $_ eq $word } $left->@* ) == 1 );
   }

   for my $word ( $right->@* ) {
       push @single_right, $word  if ( scalar( grep { $_ eq $word } $right->@* ) == 1 );
   }

   my $counter = 0;
   for my $word ( @single_left ) {
       $counter++ if ( grep { $_ eq $word } @single_right );
   }

   return $counter;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 277 - Task 2 - PL/Perl Implementation

Same implementation of the Raku solution, only more verbose because I define two internal functions `min` and `abs` to compute the conditions.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc277.task2_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   my @strong;

   my $abs = sub {
      return $_[ 0 ] * - 1 if ( $_[ 0 ] < 0 );
      return $_[ 0 ];
   };

   my $min = sub {
      return $_[ 0 ] if ( $_[ 0 ] < $_[ 1 ] );
      return $_[ 1 ];
   };

   for my $i ( 0 .. $nums->@* - 2 ) {
       for my $j ( $i + 1 .. $nums->@* - 1 ) {
       	   my ( $left, $right ) = ( $nums->@[ $i ], $nums->@[ $j ] );
		   push @strong, [ $left, $right ]  if ( $abs->( $left - $rigth ) > 0 && $abs->( $left - $right ) < $min->( $left, $right ) );
       }
   }

   return scalar( @strong );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 277 - Task 1 - PL/PgSQL Implementation


A single query can solve the problem.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc277.task1_plpgsql( words1 text[], words2 text[] )
RETURNS int
AS $CODE$

   WITH w1 AS (
   	SELECT w::text
	FROM unnest( words1 ) w
	GROUP BY w
	HAVING count(*) = 1
  ),
  w2 AS (
     	SELECT w::text
	FROM unnest( words2 ) w
	GROUP BY w
	HAVING count(*) = 1
  )
  SELECT count( l.w )
  FROM w1 l, w2 r
  WHERE l.w = r.w

$CODE$
LANGUAGE sql;

```
<br/>
<br/>

The materialzed queries `w1` and `w2` provide a list of words that appear only once in their list.
The last `SELECT` *joins* the lists to find out which words appear in both the lists, and finally a `count` provides the result.


<a name="task2plpgsql"></a>
## PWC 277 - Task 2 - PL/PgSQL Implementation


I use a nested loop, as in PL/Perl, and a query to compute the `min` value.
Note that, in order to have `abs` to be greater than zero, it does suffice to test only if the numbers are not the same.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc277.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	strong_counter int := 0;
	current int := 0;
BEGIN

	FOR i IN 1 .. array_length( nums, 1 ) - 1 LOOP
	    FOR j in ( i + 1 ) .. array_length( nums, 1 ) LOOP
	    	SELECT min( x )
			INTO current
			FROM unnest( array[ nums[ i ], nums[ j ] ] ) x;

	    	IF current > abs( nums[ i ] - nums[ j ] ) AND nums[ i ] <> nums[ j ] THEN
				strong_counter := strong_counter + 1;
		    END IF;
	    END LOOP;
	END LOOP;

	RETURN strong_counter;

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 277 - Task 1 - PostgreSQL PL/Java Implementation

I use the stream API here. With a map `counting` I store every word with two integers, one that will count the number of appeareance in the first list and the second in the second list respectively.

<br/>
<br/>
```java
    public static int task1_pljava( String[] words1, String[] words2 ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc277.task1_pljava" );

		final Map<String, Integer[]> counting = new HashMap<String, Integer[]>();

		Stream.of( words1 ).forEach( current -> {
			Integer[] count = { 0, 0 };
			counting.putIfAbsent( current, count );
			count = counting.get( current );
			count[ 0 ]++;
			counting.put( current, count );
		    } );


		Stream.of( words2 ).forEach( current -> {
			Integer[] count = { 0, 0 };
			counting.putIfAbsent( current, count );
			count = counting.get( current );
			count[ 1 ]++;
			counting.put( current, count );
		    } );

		return (int) counting.entrySet().stream().filter( current -> {
			Integer[] count = current.getValue();
			return count[ 0 ] == count[ 1 ] && count[ 0 ] == 1;
		    } ).count();
    }
```
<br/>
<br/>

The counting of appearances is done first computing a `Stream.of` from the array, then `forEach` puts into the `counting` map the array with zero values if not already there. if the entry is then in the map, I increment the cell count.

Last, I extract the `entrySet` stream filtering those that have both cells equal to `1`, `count`-ing the results and returning the value.



<a name="task2pljava"></a>
## PWC 277 - Task 2 - PostgreSQL PL/Java Implementation

Again, using the stream API, I iterate over the indexes of the `numbers` array.


<br/>
<br/>
```java
    public static final int task2_pljava( int[] numbers ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc277.task2_pljava" );

		final int[] c = new int[]{ 0 };
		IntStream.range( 0, numbers.length - 1 )
		    .forEach( i -> {
			    c[ 0 ] += IntStream.range( i + 1, numbers.length )
				.filter( j -> {
					return numbers[ i ] != numbers[ j ]
					    && Math.abs( numbers[ i ] - numbers[ j ] ) < Math.min( numbers[ i ], numbers[ j ] );
				    } ).count();
			} );

		return c[ 0 ];
    }
```
<br/>
<br/>

First, I create a stream over the indexes assigning to the value `i`, then `forEach` index I get another stream range (assigned to `j`), `filter`-ing so that the bs and `min` values satisfy the condition.



# Python Implementations

<a name="task1python"></a>
## PWC 277 - Task 1 - Python Implementation

Similar solution to PL/Perl implementation, but there is the need to separate the arrays using a pipe symbol on the command line.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    words1 = []
    words2 = []
    first  = True
    for w in args:
        if w == '|':
            first = False
        if first:
            words1.append( w )
        else:
            words2.append( w )

    single1 = []
    single2 = []
    for w in words1:
        if len( list( filter( lambda x: x == w, words1 ) ) ) == 1:
            if not w in single1:
                single1.append( w )

    for w in words2:
        if len( list( filter( lambda x: x == w, words2 ) ) ) == 1:
            if not w in single2:
                single2.append( w )

    counting = 0
    for w in single1:
        if w in single2:
            counting += 1

    return counting


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>


I keep two lists of words appearing once on their list, and then count how many matches are there.


<a name="task2python"></a>
## PWC 277 - Task 2 - Python Implementation

A nested loop on the indexes.
I append a string representing both the numbers into the `strong` array, returning then the size of the list.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    nums = list( map( int, args ) )

    strong = []

    for i in range( 0, len( nums ) - 1 ):
        for j in range( i + 1, len( nums ) ):
            if nums[ i ] != nums[ j ] and abs( nums[ i ] - nums[ j ] ) < min( nums[ i ], nums[ j ] ):
               strong.append( str( nums[ i ] ) + "<->" + str( nums[ j ] ) )

    return len( strong )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
