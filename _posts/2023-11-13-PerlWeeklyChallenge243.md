---
layout: post
title:  "Perl Weekly Challenge 243: map and grep to the rescue!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 243: map and grep to the rescue!

This post presents my solutions to the [Perl Weekly Challenge 243](https://perlweeklychallenge.org/blog/perl-weekly-challenge-243/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 243 - Task 1 - Raku](#task1)
- [PWC 243 - Task 2 - Raku](#task2)
- [PWC 243 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 243 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 243 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 243 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 243 - Task 1 in Python](#task1python)
- [PWC 243 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 243 - Task 1 - Raku Implementation

The first task was about finding *reverse pairs* in an array of integers. A reverse pair is a couple of integeres in an array where the leftmost one is greater than the double value of the rightmost one.
This can be solved in an easy way by means of a nested loop, but here `grep` can help solving the problem in a more elegant way.

<br/>
<br/>
```raku
sub MAIN( Bool :$verbose = True,
			   *@nums where { @nums.elems == @nums.grep( { $_ ~~ Int } ).elems } ) {


    # A reverse pair is a pair (i, j) where: a) 0 <= i < j < nums.length and b) nums[i] > 2 * nums[j].
    my @reverse_pairs;
    for 0 ..^ @nums.elems -> $i {
		@reverse_pairs.push: [ @nums[ $i ], $_ ] for  @nums[ $i + 1 .. * ].grep( { @nums[ $i ] > 2 * $_ } );
    }

    @reverse_pairs.elems.say;
    @reverse_pairs.join( "\n" ).say if $verbose;
}
```
<br/>
<br/>

I iterate on every element of the array using a positional index c$i, then  I slice the array as `@nums[ $i + 1 .. * ]` and `grep` it for elements smaller than the half of the current value, and push a couple of values into the `@reverse_pairs` array.
Last, I count the number of elements in the array and all it's done!


<a name="task2"></a>
## PWC 243 - Task 2 - Raku Implementation

This task is about computing the sum of the division of every element of the array for all the remaining elements, including itself.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems && @nums.elems >= 1 } ) {
    my $sum = 0;

    for @nums -> $current {
		$sum += [+] ( @nums.map( { ( $current / $_ ).Int }  ) );
    }

    $sum.say;
}

```
<br/>
<br/>

I iterate on every element of the array, storing it as `$current`, and then `map` the array into the value of the integer division, reducing it by means of the `[+]` operator.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 243 - Task 1 - PL/Perl Implementation

Same implementation as Raku: I `push` every element `grep`ped out the list of rightmost elements that satisfay the condition, and return the size of the found array.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc243.task1_plperl( int[] )
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   my @pairs;

   for my $i ( 0 .. $nums->@* ) {
       for ( grep( { $nums->[ $i ] > 2 * $_  } $nums->@[ ( $i + 1 ) .. $nums->@* ] ) ) {
           push @pairs, [ $nums->[ $i ], $_ ] if $_;
       }
   }

   return scalar @pairs;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 243 - Task 2 - PL/Perl Implementation

Again, I `map` the array with the division of the current value with the current array element. In Perl there is not a builtin reduce operator, and instead of loading a module for the sake of summing, I simply loop over all the mapped elements and sum them together.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc243.task2_plperl( int[])
RETURNS int
AS $CODE$

   my ( $nums ) = @_;
   my $sum = 0;

   for my $current ( $nums->@* ) {
       for my $value ( map { int( $current / $_ ) } $nums->@* ) {
       	   $sum += $value;
       }
   }

   return $sum;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 243 - Task 1 - PL/PgSQL Implementation

A nested loop to solve the problem. It could have been solved with a query and `count`, but it is simple enough to be implemented in an imperative way.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc243.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	c int := 0;
BEGIN
	FOR i IN 1 .. array_length( nums, 1 ) LOOP
	    FOR j IN i + 1 .. array_length( nums, 1 ) LOOP
	    	IF nums[ i ] > nums[ j ] * 2 THEN
				c := c + 1;
			END IF;
	    END LOOP;
	END LOOP;

	RETURN c;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 243 - Task 2 - PL/PgSQL Implementation

Again, a nested loop to compute the sum, similarly to the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc243.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	s int := 0;
	current int;
BEGIN
	FOR i in 1 .. array_length( nums, 1 ) LOOP

	    FOREACH current IN ARRAY nums LOOP
	    	    s := s + ( nums[ i ] / current )::int;
	    END LOOP;
	END LOOP;

	RETURN s;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 243 - Task 1 - Python Implementation

A nested loop to solve the problem.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    nums = list( map( int, argv ) )
    counter = 0

    for i in range( 0, len( nums ) ):
        for j in range( i + 1, len( nums ) ):
            if nums[ i ] > 2 * nums[ j ]:
                counter = counter + 1;

    print( counter )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 243 - Task 2 - Python Implementation

To be more *Perl-ish* I use a double `lambda` expression to `map` and `reduce` the array on which I need to compute the current sum.

<br/>
<br/>
```python
import sys
from functools import reduce

# task implementation
def main( argv ):
    nums = list( map( int, argv ) )
    sum = 0


    for current in nums:
        sum = sum + reduce( lambda a,b: a + b, list( map( lambda x: int( current /x ), nums ) ) )

    print( sum )

# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>
