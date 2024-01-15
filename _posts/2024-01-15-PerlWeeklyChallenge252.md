---
layout: post
title:  "Perl Weekly Challenge 252: doing math to get zero!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 252: doing math to get zero!

This post presents my solutions to the [Perl Weekly Challenge 252](https://perlweeklychallenge.org/blog/perl-weekly-challenge-252/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 252 - Task 1 - Raku](#task1)
- [PWC 252 - Task 2 - Raku](#task2)
- [PWC 252 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 252 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 252 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 252 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 252 - Task 1 in Python](#task1python)
- [PWC 252 - Task 2 in Python](#task2python)
- [PWC 252 - Task 1 in Java](#task1java)
- [PWC 252 - Task 2 in Java](#task2java)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 252 - Task 1 - Raku Implementation

The first task was about summing the squares of *special numbers* within a given array. A special number is the one indexed so that the length of the array modulus the position of the number is good.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    @nums.kv.map( { @nums.elems %% ( $^index + 1 ) ?? $^value ** 2 !! 0  } ).sum.say;
}

```
<br/>
<br/>

Here I use a little trick: `@nums.kv` returns a list of pairs, namely `$^index` and `$^value` with the value and indexe within the array.
Therefore, I'm able to `map` to get a single line program.


<a name="task2"></a>
## PWC 252 - Task 2 - Raku Implementation

The second task was about generating an array of the given size, so that the sum of all the elements drops to zero.

<br/>
<br/>
```raku
sub MAIN( $n where { $n >= 3 && $n ~~ Int  } ) {
    my @nums;

    if $n %% 2 {
		@nums.push: $_, $_ * -1 for 1 .. ( $n / 2 );
    }
    else {
		@nums.push: $_, $_ * -1 for 1 .. ( ( $n - 2 ) / 2 );
		my $next = @nums.max + 1;
		@nums.push: $next, $next + 1, ( $next + $next + 1 ) * -1;
    }

    @nums.join( ',' ).say;
}

```
<br/>
<br/>

The idea is that if the requested size is even, then I only need to add couple `i` and `-i`, so that all the sums are zero.
If the requested size is odd, then I do the same antipairs addition, but keeps the last three elements of the array to fill. I fille two of the remainings with incremented integers, and the last one with the negative sum of the last twos.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 252 - Task 1 - PL/Perl Implementation

Here I thought to use `map` against `each`, but this does not work as I was expecting, so I had to do a loop instead.



<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc252.task1_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $nums ) = @_;

   my ( $sum ) = 0;

   for ( 0 .. $nums->@* - 1 ) {
       next if $nums->@* % ( $_ + 1 ) != 0;
       $sum += $nums->@[ $_ ] ** 2;
   }

   return( $sum );
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 252 - Task 2 - PL/Perl Implementation

The solution is the same as in Raku: I distinguish the odd/even cases.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc252.task2_plperl( int )
RETURNS SETOF int
AS $CODE$
   my ( $size ) = @_;

   die "Cannot have a size less than 3!" if ( $size <= 3 );

   if ( $size % 2 == 0 ) {
      for ( 1 .. $size / 2 ) {
      	  return_next( $_ );
	  return_next( $_ * -1 );
      }
   }
   else {
       for ( 1 .. ( $size - 1 ) / 2 ) {
      	  return_next( $_ );
	  return_next( $_ * -1 );
      }

      my $next = int( $size / 2 ) + 1;
      return_next( $next );
      return_next( $next + 1 );
      return_next( ( $next + $next + 1 ) * -1 );
   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 252 - Task 1 - PL/PgSQL Implementation

The same approach as PL/Perl: I iterate over the array and keep track only of interesting numbers.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc252.task1_plpgsql( nums int[] )
RETURNS int
AS $CODE$
DECLARE
	sumx int := 0;
BEGIN

	FOR i IN 1 .. array_length( nums, 1 ) LOOP
	    IF mod( array_length( nums, 1 ), i ) <> 0 THEN
	       CONTINUE;
	    END IF;

	    sumx := sumx + pow( nums[ i ], 2 );
	END LOOP;

	RETURN sumx;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 252 - Task 2 - PL/PgSQL Implementation

The approach is yet the same as in Raku.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc252.task2_plpgsql( size int )
RETURNS SETOF int
AS $CODE$
DECLARE
	next_value int;
BEGIN

	IF size <= 3 THEN
	   RAISE 'Cannot work with a size less than 3!';
	END IF;

	IF mod( size, 2 ) = 0 THEN
	   FOR i IN 1 .. ( size / 2 ) LOOP
	       RETURN NEXT i;
	       RETURN NEXT (i * -1);
	   END LOOP;
	ELSE
	    FOR i IN 2 .. ( size - 1 ) / 2  LOOP
	       RETURN NEXT i;
	       RETURN NEXT (i * -1);
	   END LOOP;

	   next_value := ( size - 1 ) / 2  + 1;
	   RETURN NEXT next_value;
	   RETURN NEXT next_value + 1;
	   RETURN NEXT ( next_value + next_value + 1 ) * -1;
	END IF;
RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 252 - Task 1 - Python Implementation

Same approach as in PL/Perl.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def main( argv ):
    nums = list( map( int, argv ) )
    sum = 0
    for i in range( 0, len( nums ) ):
        if len( nums ) % ( i + 1 ) == 0:
            sum += nums[ i ] ** 2;

    return sum


# invoke the main without the command itself
if __name__ == '__main__':
    print( main( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 252 - Task 2 - Python Implementation

Again, same approach as in Raku and other implementations.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def main( argv ):
    size = int( argv[ 0 ] )
    nums = []

    if size % 2 == 0:
        for i in range( 1, size / 2 ):
            nums.append( i )
            nums.append( i * -1 )
    else:
        for i in range( 1, int( ( size - 1 ) / 2 ) ):
            nums.append( i )
            nums.append( i * -1 )

        next = max( nums ) + 1
        nums.append( next )
        nums.append( next + 1 )
        nums.append( ( next + next + 1 ) * -1 )

    return nums



# invoke the main without the command itself
if __name__ == '__main__':
    print( main( sys.argv[ 1: ] ) )


```
<br/>
<br/>


# Java Implementations

<a name="task1java"></a>
## PWC 252 - Task 1 - Java Implementation

Same iterating approach as in PL/Perl.

<br/>
<br/>
```java
public class ch_1 {
    public static void main( String argv[] ) {
	int sum = 0;
	for ( int i = 0; i < argv.length; i++ ) {
	    if ( argv.length % ( i + 1 ) != 0 )
		continue;

	    sum += Math.pow( Integer.parseInt( argv[ i ] ), 2 );
	}

	System.out.println( sum );
    }
}

```
<br/>
<br/>



<a name="task2java"></a>
## PWC 252 - Task 2 - Java Implementation

Again, the very same approach I used in all other implementations.


<br/>
<br/>
```java
public class ch_2 {
    public static void main( String argv[] ) throws Exception {
	int size = Integer.parseInt( argv[ 0 ] );

	if ( size <= 3 )
	    throw new Exception( "Cannot work with a size less than 3!" );

	List<Integer> nums = new LinkedList<Integer>();


	if ( size % 2 == 0 ) {
	    for ( int i = 1; i < size / 2 ; i++ ) {
		nums.add( i );
		nums.add( i * -1 );
	    }
	}
	else {
	    for ( int i = 1; i < ( size - 1 ) / 2 ; i++ ) {
		nums.add( i );
		nums.add( i * -1 );
	    }

	    int next = ( size - 1 ) / 2 + 1;
	    nums.add( next );
	    nums.add( next + 1 );
	    nums.add( ( next * 2 + 1 ) * -1 );
	}


	boolean printedOne = false;
	for ( int i : nums ) {
	    System.out.print( i + ( printedOne ? ", " : "" ) );
	    printedOne = true;
	}

	System.out.println();
    }
}

```
<br/>
<br/>
