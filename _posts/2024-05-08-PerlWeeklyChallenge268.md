---
layout: post
title:  "Perl Weekly Challenge 268: arrays and slices"
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

# Perl Weekly Challenge 268: arrays and slices

This post presents my solutions to the [Perl Weekly Challenge 268](https://perlweeklychallenge.org/blog/perl-weekly-challenge-268/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 268 - Task 1 - Raku](#task1)
- [PWC 268 - Task 2 - Raku](#task2)
- [PWC 268 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 268 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 268 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 268 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 268 - Task 1 in PL/Java](#task1pljava)
- [PWC 268 - Task 2 in PL/Java](#task2pljava)
- [PWC 268 - Task 1 in Python](#task1python)
- [PWC 268 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 268 - Task 1 - Raku Implementation

The first task was to find the *magic number* between two equally sized arrays of integers. The magic n umber is the difference between two elements of the arrays so that from the first array you can obtain the second one, even if in a different order.
Essentially, it means that the sorted arrays have values with a consistent difference.

<br/>
<br/>
```raku
sub MAIN(  :@left,  :@right where { @left.elems == @right.elems }  ) {
    my @sorted-left  = @left.sort;
    my @sorted-right = @right.sort;

    my @diffs;
    for 0 ..^ @sorted-left.elems -> $i {
		my $current = @sorted-left[ $i ] - @sorted-right[ $i ];
		@diffs.push: $current if ! @diffs.grep( $current );
    }

    @diffs[ 0 ].say and exit  if @diffs.elems == 1;
    'Impossible'.say;
}

```
<br/>
<br/>

I first sort the arrays, then I iterate and compute the `$current` difference between sorted cells, and add it the list of `@diffs` if not already inserted. In the end, if the list of `@diffs` has only one element, that is the magic number, otherwise the magic number does not exist.


<a name="task2"></a>
## PWC 268 - Task 2 - Raku Implementation

The second task was about sorting a given input array of integers by sorting its two elements slices.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems  && @nums.elems %% 2 }  ) {
    my @sorted;
    for @nums.sort.rotor( 2 ) -> @couple {
		@sorted.push: @couple.reverse.flat;
    }

    @sorted.join( " " ).say;
}

```
<br/>
<br/>

I use the `rotor` that automatically splits the array in sub elements of two parts, and sort the subarray in the way required, then pushing as a flat list into the output result `@sorted`.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 268 - Task 1 - PL/Perl Implementation

The same implementation as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc268.task1_plperl( int[], int[] )
RETURNS int
AS $CODE$

   my ( $left, $right ) = @_;

   die 'Array must have the same length!' if ( $left->@* != $right->@* );

   my @diffs;
   for my $index ( 0 .. $left->@* - 1 ) {
       my $d = $left->@[ $index ] - $right->@[ $index ];
       push @diffs, $d if ( ! grep( { $_ == $d } @diffs ) );
   }

   return $diffs[ 0 ] if ( @diffs == 1 );
   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 268 - Task 2 - PL/Perl Implementation

Same implementtion as in Raku, but since I don't have the `rotor`, I extract the couple of elements and sort them manually.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc268.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $nums ) = @_;

   $nums = [ sort $nums->@* ];
   my @sorted;
   for my $index ( 0 .. $nums->@* - 1 ) {
       next if $index % 2 != 0;
       push @sorted, reverse sort( $nums->@[ $index ], $nums->@[ $index + 1 ] );
   }

   return [ @sorted ]	;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 268 - Task 1 - PL/PgSQL Implementation

A single query, with a CTE, can solve the problem.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc268.task1_plpgsql( l int[], r int[] )
RETURNS int
AS $CODE$

   WITH sorted_left AS ( SELECT v, row_number()  OVER ( order by 1 ) as r  FROM unnest( l ) v  )
   , sorted_right AS ( SELECT v, row_number()  OVER ( order by 1 ) as r  FROM unnest( r ) v  )
   , diffs AS (
    SELECT l.v - r.v as d
    FROM sorted_left l, sorted_right r
    WHERE l.r = r.r
   )
   SELECT d
   FROM diffs d
   GROUP BY d
   HAVING count(*) = ( SELECT array_length( l, 1 ) );


$CODE$
LANGUAGE sql;

```
<br/>
<br/>

I use the `row_number()` function to get the position of every sorted element, so that I can compute the differences between elements with the same index (i.e., position), and then I select only the difference that appears the exact number as the length of the array.



<a name="task2plpgsql"></a>
## PWC 268 - Task 2 - PL/PgSQL Implementation

Here I cheated, I call the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc268.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
   SELECT pwc268.task2_plperl( nums );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 268 - Task 1 - PostgreSQL PL/Java Implementation

The idea is the same as in PL/Perl approach: I sort the lists of integers and compute all the differences, returning the found value if there is only one.

<br/>
<br/>
```java
public class Task1 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc268",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final Integer task1_pljava( int[] left, int[] right ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc268.task1_pljava" );

		List<Integer> l = new LinkedList<Integer>();
		List<Integer> r = new LinkedList<Integer>();

		for ( int i : left )
		    l.add( i );
		for ( int i : right )
		    r.add( i );

		Collections.sort( l );
		Collections.sort( r );
		List<Integer> diffs = new LinkedList<Integer>();

		for ( int i = 0; i < l.size(); i++ ) {
		    int current = l.get( i ) - r.get( i );
		    if ( ! diffs.contains( current ) )
				diffs.add( current );
		}

		if ( diffs.size() == 1 )
		    return diffs.get( 0 );
		else
		    return null;


    }
}

```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 268 - Task 2 - PostgreSQL PL/Java Implementation

Same approach as in PL/Perl: extract the single couple of elements of every slice and manually sort them by means of the ternary operator.

<br/>
<br/>
```java
public class Task2 {

    private final static Logger logger = Logger.getAnonymousLogger();

    @Function( schema = "pwc268",
	       onNullInput = RETURNS_NULL,
	       effects = IMMUTABLE )
    public static final int[] task2_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc268.task2_pljava" );

		List<Integer> sorted = new LinkedList<Integer>();
		for ( int i : nums )
		    sorted.add( i );
		Collections.sort( nums );

		int[] result = new int[ nums.length ];
		int index = 0;

		for ( int i = 0; i < sorted.size(); i++ ) {
		    int left = sorted.get( i );
		    int right = sorted.get( ++i );

		    result[ index++ ]   = ( left > right ? left : right );
		    result[ index++ ] = ( left > right ? right : left );
		}

		return result;
    }
}

```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 268 - Task 1 - Python Implementation

Same approach as in PL/Perl, but since there is no common way to pass two different arrays on the command line, I use the `|` as a separator.

<br/>
<br/>
```python
def task_1( args ):
    left = []
    right = []

    left  = list( map( int, args[ 0 : args.index( "|" ) ] ) )
    right = list( map( int, args[ args.index( "|" ) + 1 : ] ) )

    left.sort()
    right.sort()

    diffs = []
    for index in range( 0, len( left ) ):
        d = left[ index ] - right[ index ]
        if not d in diffs:
            diffs.append( d )

    if len( diffs ) == 1:
        return diffs[ 0 ]
    else:
        return None



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 268 - Task 2 - Python Implementation

Same approach already implemented in PL/Java.

<br/>
<br/>
```python
def task_2( args ):
    sorted = list( map( int, args ) )
    sorted.sort()

    result = []

    for i in range( 0, len( sorted ) ):
        if i % 2 != 0:
            continue

        left  = sorted[ i ]
        right = sorted[ i + 1 ]

        result.append( left if left > right else right )
        result.append( left if right > left else right )

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
