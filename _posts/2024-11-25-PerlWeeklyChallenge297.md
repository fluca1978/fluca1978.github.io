---
layout: post
title:  "Perl Weekly Challenge 297: filtering arrays"
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

# Perl Weekly Challenge 297: filtering arrays

This post presents my solutions to the [Perl Weekly Challenge 297](https://perlweeklychallenge.org/blog/perl-weekly-challenge-297/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 297 - Task 1 - Raku](#task1)
- [PWC 297 - Task 2 - Raku](#task2)
- [PWC 297 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 297 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 297 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 297 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 297 - Task 1 in PL/Java](#task1pljava)
- [PWC 297 - Task 2 in PL/Java](#task2pljava)
- [PWC 297 - Task 1 in Python](#task1python)
- [PWC 297 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 297 - Task 1 - Raku Implementation

The first task was to find out the max length of an array of consecutive bits, given as input, so that the array is balanced in the number of ones and zeros.

<br/>
<br/>
```raku
sub MAIN( *@bits where { @bits.grep( * !~~ any(0,1) ).elems == 0 } ) {
    my $max-length = 0;

    for 0 ..^ @bits.elems - 1 -> $start {
		for $start ^..^ @bits.elems -> $end {
			next if @bits.[ $start .. $end ].grep( * ~~ 0 ).elems != @bits[ $start .. $end ].grep( * ~~ 1 ).elems;
			$max-length = $end - $start + 1 if ( $end - $start + 1 > $max-length );
		}
    }

    $max-length.say;
}

```
<br/>
<br/>


My approach was to consecutively build slices, made by `$start` and `$end` indexes, and to count if the number of ones and zeros in the slice are balanced. In the case they are, check if the length is greater than the previously computed one, so to accumulate only the max one.



<a name="task2"></a>
## PWC 297 - Task 2 - Raku Implementation

The second task was to find out, given an array of unique numbers, how many *swaps* has to be performed to move the `1` to the leftmost position and the value equal to the array length to the rightmost one.


<br/>
<br/>
```raku
sub MAIN( *@ints is copy where { @ints.grep( *.Int ).elems == @ints.elems } ) {
    my $swaps = 0;
    my $index =  @ints.grep( * == 1, :k ).first;
    $swaps += $index;

    @ints = | 1, | @ints[ 0 ..^ $index ], | @ints[ $index ^.. * - 1 ];

    $index =  @ints.grep( * == @ints.elems, :k ).first;
    $swaps += @ints.elems - $index - 1;

    @ints = | @ints[ 0 ..^ $index ], | @ints[ $index ^.. * - 1 ], @ints.elems;

    $swaps.say;

}

```
<br/>
<br/>

With regard to moving the `1` to the leftmost position, well, it does suffice to compute the index of the number and that is the number of hops, or swaps, it has to perform.
Then the array is different, since all other digits have been moved to the right, so I rebuild it by concatenating the slices, and then compute the number of hopes that the highest number has to perform to go the rightmost position.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 297 - Task 1 - PL/Perl Implementation

The same implementation as in Raku, only with a different verbosity.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc297.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $bits ) = @_;
   my $max_length = 0;

   die "Only bits are allowed!" if ( grep( { $_ != 0 && $_ != 1 } $bits->@* ) );

   for my $start ( 0 .. $bits->@* - 2 ) {
       for my $end ( $start + 1 .. $bits->@* - 1 ) {
       	   my $zeros = grep { $_ == 0 } $bits->@[ $start .. $end ];
		   my $ones  = grep { $_ == 1 } $bits->@[ $start .. $end ];
		   next if ( $ones != $zeros );

		   my $length = $end - $start + 1;
		   $max_length = $length if ( $length > $max_length );
       }
   }

   return $max_length;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 297 - Task 2 - PL/Perl Implementation

Same implementation as in Raku, but a lot more verbose.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc297.task2_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $ints ) = @_;

   my $swaps = 0;
   my $index = 0;

   my $current_index = (
   			map { $_->[ 1 ] }
   			grep { $_->[ 0 ] == 1 }
   			map { [ $ints->[ $_ ], $_ ] } 0 .. $ints->@* - 1 )[ 0 ];

  $ints = [ 1, $ints->@[ 0 .. $current_index - 1 ],
               $ints->@[ $current_index + 1 .. -1 ] ];
  $swaps += $current_index;


  $current_index = (
   			map { $_->[ 1 ] }
   			grep { $_->[ 0 ] == scalar $ints->@* }
   			map { [ $ints[ $_ ], $_ ] } 0 .. $ints->@* - 1 )[ 0 ];

  $ints = [ $ints->@[ 0 .. $current_index - 1 ],
            $ints->@[ $current_index + 1 .. -1 ],
	    scalar $ints->@* ];

  $swaps +=  scalar( $ints->@* ) - $current_index - 1;
  return $swaps;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 297 - Task 1 - PL/PgSQL Implementation

The idea is the same as in PL/Perl.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc297.task1_plpgsql( bits bit[] )
RETURNS int
AS $CODE$
DECLARE
	i_begin int;
	i_end   int;
	max_length int := 0;
	zeros int;
	ones  int;
	current_length int;
BEGIN

	for i_begin in 1 .. array_length( bits, 1 ) - 1 loop
	    for i_end in i_begin + 1 .. array_length( bits, 1 ) loop

	    	current_length := i_end - i_begin + 1;

	    	SELECT count( v )
		INTO ones
		FROM unnest( bits[ i_begin : i_end ] ) v
		WHERE v = 1::bit;

	    	SELECT count( v )
		INTO zeros
		FROM unnest( bits[ i_begin : i_end ] ) v
		WHERE v = 0::bit;

		IF zeros <> ones OR current_length < max_length THEN
		   continue;
		END IF;

		max_length := current_length;


	    end loop;
	end loop;

	return max_length;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

One difference is that, in this case, I test for a single condition about the number of ones and zeros and, at the same time, the length.


<a name="task2plpgsql"></a>
## PWC 297 - Task 2 - PL/PgSQL Implementation

A single query does suffice.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc297.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
	with sorted as (
	     select v::int, row_number() over () as i
	     from unnest( nums ) v
	)
	select f.i - 1 + array_length( nums, 1 ) - l.i
	FROM sorted f, sorted l
	WHERE
	f.v = 1
	and l.v = array_length( nums, 1 )
	;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 297 - Task 1 - PostgreSQL PL/Java Implementation

I use the Stream API to filter the array and count the ones and the zeroes, pretty much like in other implementations.


<br/>
<br/>
```java
    public static final int task1_pljava( int[] bits ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc297.task1_pljava" );

	    int max_length = 0;
	    int current_length;

	   for ( int start = 0; start < bits.length - 2; start++ ) {
	    for ( int end = start + 1; end < bits.length - 1; end++ ) {
	  	  long ones  = Arrays.stream( bits, start, end + 1 ).filter( b -> b == 1 ).count();
		  long zeros = Arrays.stream( bits, start, end + 1 ).filter( b -> b == 0 ).count();
		  current_length = end - start + 1;

		   if ( zeros == ones && current_length > max_length )
		    max_length = current_length;
	    }
	 }

	return max_length;
  }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 297 - Task 2 - PostgreSQL PL/Java Implementation

Again usage of Stream API, but with a difference with the other implementations.


<br/>
<br/>
```java
    public static final int task2_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc297.task2_pljava" );

	   int swaps = 0;
	   int f_index = IntStream.range( 0, nums.length )
	       .filter( i -> nums[ i ] == 1 )
	       .mapToObj( i -> new Integer( i ) )
	       .collect( Collectors.toList() )
	       .get( 0 );

	   swaps += f_index;

	   int l_index = IntStream.range( 0, nums.length )
	       .filter( i -> nums[ i ] == nums.length )
	       .mapToObj( i -> new Integer( i ) )
	       .collect( Collectors.toList() )
	       .get( 0 );

	   if ( f_index < l_index )
	       swaps += nums.length - l_index;
	   else
	       swaps += nums.length - l_index - ( f_index - l_index );

	   return swaps;
    }
```
<br/>
<br/>

This time I don't rebuild the array after the first moves of `1`, so I consider the two indexes and, in case there is overlapping, I consider the difference between the indexes as already done in the first move.

# Python Implementations

<a name="task1python"></a>
## PWC 297 - Task 1 - Python Implementation

Same implementation as in PL/Perl.

<br/>
<br/>
```pythonimport sys

# task implementation
# the return value will be printed
def task_1( args ):
    bits = list( map( int, args ) )
    max_length = 0

    for start in range( 0, len( bits ) - 1 ):
        for end in range( start + 1 , len( bits ) ):
            current_length = end - start + 1
            ones  = len( list( filter( lambda x: x == 1, bits[ start : end ] ) ) )
            zeros = len( list( filter( lambda x: x == 0, bits[ start : end ] ) ) )

            if ones == zeros and current_length > max_length:
                max_length = current_length

    return max_length



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )


```
<br/>
<br/>



<a name="task2python"></a>
## PWC 297 - Task 2 - Python Implementation


Same implementation as in PL/Perl, this time a little less verbose.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    nums = list( map( int, args ) )
    swaps = 0

    f_index = int( list( filter( lambda i : nums[ i ] == 1, range( 0, len( nums ) ) ) )[ 0 ] )
    l_index = int( list( filter( lambda i : nums[ i ] == len( nums ), range( 0, len( nums ) ) ) )[ 0 ] )

    swaps += f_index
    if f_index < l_index:
        swaps += len( nums ) - l_index
    else:
        swaps += len( nums ) - l_index - ( f_index - l_index )

    return swaps - 1


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
