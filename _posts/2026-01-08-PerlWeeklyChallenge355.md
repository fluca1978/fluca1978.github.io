---
layout: post
title:  "Perl Weekly Challenge 355: number formatting and sorting"
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

# Perl Weekly Challenge 355: number formatting and sorting

This post presents my solutions to the [Perl Weekly Challenge 355](https://perlweeklychallenge.org/blog/perl-weekly-challenge-355/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 355 - Task 1 - Raku](#task1)
- [PWC 355 - Task 2 - Raku](#task2)
- [PWC 355 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 355 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 355 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 355 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 355 - Task 1 in PL/Java](#task1pljava)
- [PWC 355 - Task 2 in PL/Java](#task2pljava)
- [PWC 355 - Task 1 in Python](#task1python)
- [PWC 355 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 355 - Task 1 - Raku Implementation

The first task was about printing out a number using a dot as thousand separator. This can be quickly solved if the language provide a good enough way of formatting numbers.
Since I'm on an old version of Raku, I used a one-liner to do the trick:

<br/>
<br/>
```raku
sub MAIN( Int $number ) {
    $number.Str.flip.comb.rotor( 3, :partial ).map( *.join ).join( '.' ).flip.say;
}

```
<br/>
<br/>

I split the number into its digits, revert them and get groups of `3` digits, that then I join into single groups joined again with a dot and finally I reverse the string.


<a name="task2"></a>
## PWC 355 - Task 2 - Raku Implementation


The second task was about finding out if a given array of integer is a mountain, that is it has a ascending order for the left part and a descending right part.

<br/>
<br/>
```raku
sub MAIN( *@ints where { @ints.grep( * ~~ Int ).elems == @ints.elems } ) {
    'False'.say and exit if ( @ints.elems < 3 );
    my $ok = False;
    for 1 ..^ @ints.elems -> $i {
		$ok =  @ints[ 0 .. $i ].sort.Array eqv @ints[ 0 .. $i ].Array
			&& @ints[ $i .. * - 1 ].sort( { $^b <=> $^a } ).Array eqv @ints[ $i .. * - 1 ].Array;
		last if ( $ok );
    }

    $ok.say;
}

```
<br/>
<br/>


The trick here is to compare the leftmost array with the sorted version of itself, and the rightmost array with the reverse sorted copy of itself.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 355 - Task 1 - PL/Perl Implementation


A version similar to the Raku implementation, where I split into digits, and join them using groups of three.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc355.task1_plperl( int )
RETURNS text
AS $CODE$

   my ( $number ) = @_;
   my $digits = [ reverse split //, $number ];
   my $result = "";
   for my $i ( 0 .. $digits->@* - 1 ) {
       $result .= '.' if ( $i > 0 && $i % 3 == 0 );
       $result .= $digits->@[ $i ];
   }

   return reverse $result;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 355 - Task 2 - PL/Perl Implementation

Same approach as in Raku, but I need my own `eqv` implementation, so I define a function called `cmp` to compare two arrays.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc355.task2_plperl( int[] )
RETURNS boolean
AS $CODE$

   my ( $numbers ) = @_;

   my $cmp = sub {
      my ( $left, $right ) = @_;
      return 0 if ( $left->@* != $right->@* );

      for my $i ( 0 .. $left->@* - 1 ) {
      	  return 0 if ( $left->@[ $i ] != $right->@[ $i ] );
      }

      return 1;
   };

   for my $i ( 1 .. $numbers->@* - 2 ) {
       return 1 if ( $cmp->( [ $numbers->@[ 0 .. $i ] ],
       	      	     	     [ sort $numbers->@[ 0 .. $i ] ] )
	             && $cmp->( [ $numbers->@[ $i .. $numbers->@* - 1 ] ],
		                [ sort { $b <=> $a } $numbers->@[ $i .. $numbers->@* - 1 ] ] ) );
   }

   return 0;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 355 - Task 1 - PL/PgSQL Implementation

I use a single query with a format to specify the thousand separator, replacing commas with a dot (depending on the locale).


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc355.task1_plpgsql( n int )
RETURNS text
AS $CODE$
   SELECT regexp_replace( to_char( n,
   	  	   	  	   'FM999,999,999' ),
			 ',',
			 '.',
			 'g' );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 355 - Task 2 - PL/PgSQL Implementation

I cheat this and call PL/Perl implementation!


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc355.task2_plpgsql( v int[] )
RETURNS boolean
AS $CODE$
   SELECT pwc355.task2_plperl( v );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 355 - Task 1 - PostgreSQL PL/Java Implementation

A good hammering on string formatting can do the trick.

<br/>
<br/>
```java
public static final String task1_pljava( int num ) throws SQLException {
		return String.format( "%,d", num ).replaceAll( ",", "." );
}
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 355 - Task 2 - PostgreSQL PL/Java Implementation


Similar to the PL/Perl approach, I define my own `cmp` method and use it with the leftmost and rightmost part of the array slices.
The problem here is that sorting an array in Java is an in-place operation, so I need to define the arrays  and sort them before comparing, instead of doing it in a streamline manner.

<br/>
<br/>
```java
    public static final boolean task2_pljava( Integer[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc355.task2_pljava" );

		boolean ok = false;

		for ( int i = 1; i < nums.length; i++ ) {
		    Integer[] left  = Arrays.copyOfRange( nums, 0, i );
		    Integer[] left2 = Arrays.copyOfRange( nums, 0, i );
		    Arrays.sort( left2 );
		    Integer[] right  = Arrays.copyOfRange( nums, i, nums.length );
		    Integer[] right2 = Arrays.copyOfRange( nums, i, nums.length );
		    Arrays.sort( right2, Collections.reverseOrder() );

		    ok = cmp( left, left2 ) && cmp( right, right2 );

		    if ( ok )
				break;
		}

		return ok;
    }


    private static final boolean cmp( Integer[] left, Integer[] right ) {
		if ( left.length != right.length )
		    return false;

		for ( int i = 0; i < left.length; i++ )
		    if ( left[ i ] != right[ i ] )
			return false;

		return true;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 355 - Task 1 - Python Implementation

A very short one line implementation.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    return '{:,}'.format( int( args[ 0 ] ) ).replace( ',', '.' )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 355 - Task 2 - Python Implementation

Same approach as in PL/Perl, splice the array and define a comparator-ish function.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    nums = list( map( int, args ) )

    ok = False
    for i in range( 1, len( nums ) - 1 ):
        left  = nums[ 0 : i ]
        left2 = nums[ 0 : i ]
        left2.sort()
        right  = nums[ i : ]
        right2 = nums[ i : ]
        right2.sort( reverse = True )

        if cmp( left, left2 ) and cmp( right, right2 ):
            return True

    return False

def cmp( left, right ):
    if len( left ) != len( right ):
        return False

    for i in range(0, len( left ) ):
        if left[ i ] != right[ i ]:
            return False

    return True

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
