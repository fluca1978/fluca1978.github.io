---
layout: post
title:  "Perl Weekly Challenge 241: loops everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 241: loops everywhere!

This post presents my solutions to the [Perl Weekly Challenge 241](https://perlweeklychallenge.org/blog/perl-weekly-challenge-241/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 241 - Task 1 - Raku](#task1)
- [PWC 241 - Task 2 - Raku](#task2)
- [PWC 241 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 241 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 241 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 241 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 241 - Task 1 in Python](#task1python)
- [PWC 241 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 241 - Task 1 - Raku Implementation

The first task was about finding a triplet in an already sorted array where the difference between couple of values is the searched for value.
I implemented this with a straighforward nested triple nested loop.

<br/>
<br/>
```raku
sub MAIN( Int :$diff
	  , Bool :$verbose = True
	  , *@nums where { @nums.grep( * ~~ Int ).Array.elems == @nums.elems }
	) {

    my @triplets;
    # a) i < j < k
    # b) nums[j] - nums[i] == diff
    # c) nums[k] - nums[j] == diff
    for 0 ..^ @nums.elems -> $i {
		for $i ^..^ @nums.elems -> $j {
		    for $j ^..^ @nums.elems -> $k {
				@triplets.push: [ @nums[ $k, $j, $i ] ] if ( ( @nums[ $j ] - @nums[ $i ] ) == ( @nums[ $k ] - @nums[ $j ] )
									   && ( @nums[ $k ] - @nums[ $j ] ) == $diff );
			    }
		}
    }


    @triplets.elems.say;
    @triplets.join( "\n" ).say;

}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 241 - Task 2 - Raku Implementation

The second task was about sorting input numbers by the count of their prime factors.
I needed an helper function to compute the list of prime factors, then I stored every number into an hash keyed by the count of the prime factor list, so that sorting the hash by key gives me back the sorted values.

<br/>
<br/>
```raku
sub prime_factors (Int $n) {
    return $n if $n <= 1;
    my $residue = $n;
    my @factors;

    for 2 .. $n {
	while ( $residue %% $_ ) {
	    @factors.push: $_;
	    $residue /= $_;
	}

	last if $residue == 1;
    }

    return @factors;
}

sub MAIN( *@nums where { @nums.grep( * ~~Int ).elems == @nums.elems } ) {
    my %sorted;
    for @nums {
	%sorted{ prime_factors( $_.Int ).elems }.push: $_;
    }

    %sorted{ $_ }.join( "," ).say for  %sorted.keys.sort;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 241 - Task 1 - PL/Perl Implementation

The same implementation already exolained, with a triple nested loop.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc241.task1_plperl( int[], int )
RETURNS int
AS $CODE$
   my ( $nums, $diff ) = @_;
   my @triplets;

   for my $i ( 0 .. $nums->@* - 1) {
       for my $j ( $i + 1 .. $nums->@* - 1) {
       	   for my $k ( $j + 1 .. $nums->@* - 1) {
	       if ( ( $nums->[ $k ] - $nums->[ $j ] ) == ( $nums->[ $j ] - $nums->[ $i ] )
	       	  && ( $nums->[ $k ] - $nums->[ $j ] ) == $diff ) {
		  push @triplets, [ $nums->[ $k ], $nums->[ $j ], $nums->[ $i ] ];
	      }
	   }
       }
   }

   return scalar @triplets;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 241 - Task 2 - PL/Perl Implementation

Same implementation as Raku, but this time I use an inner function to get the primes.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc241.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   my $factors = sub {
      my ( $n ) = @_;
      my $primes = [];

      for ( 2 .. $n ) {
      	  while ( $n % $_ == 0 ) {
	  	push $primes->@*, $_;
		$n /= $_;
	  }

	  last if $n == 1;
      }

      return $primes;
   };

   my $sorted = {};
   push $sorted->{ scalar( $factors->( $_ )->@* ) }->@*, $_ for ( $nums->@* );

   for my $key ( sort keys $sorted->%* ) {
       return_next( $_ ) for ( $sorted->{ $key }->@* );
   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 241 - Task 1 - PL/PgSQL Implementation

Same triple nested loop as per other implementations:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc241.task1_plpgsql( nums int[], diff int )
RETURNS int
AS $CODE$
DECLARE
	counter int := 0;
BEGIN
	FOR i IN 1 .. array_length( nums, 1 ) LOOP
	    FOR j IN ( i + 1 ) .. array_length( nums, 1 ) LOOP
	    	FOR k IN ( j + 1 ) .. array_length( nums, 1 ) LOOP
		    IF ( nums[ k ] - nums[ j ] ) = ( nums[ j ] - nums[ i ] ) AND ( nums[ k ] - nums[ j ] = diff ) THEN
		       counter := counter + 1;
		    END IF;
		END LOOP;
	    END LOOP;
	END LOOP;

	RETURN counter;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 241 - Task 2 - PL/PgSQL Implementation

This time I use a temporary table to store the number and its prime factor counting. Then I query the table to get the values sorted by the counting of the primes.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc241.primes( n int )
RETURNS SETOF int
AS $CODE$
BEGIN
	FOR i IN 2 .. n LOOP
	    WHILE ( n % i ) = 0 LOOP
	    	  RETURN NEXT i;
		  n := n / i;
	    END LOOP;


	    IF n = 1 THEN
	       RETURN;
	    END IF;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION
pwc241.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	v int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS sorted( n int, p int DEFAULT 0 );
	TRUNCATE sorted;

	FOREACH v IN ARRAY nums LOOP
		INSERT INTO sorted( n, p )
		SELECT v, pp
		FROM pwc241.primes( v ) pp;
	END LOOP;

	RETURN QUERY
	       WITH q( n ) AS ( SELECT n FROM sorted ORDER BY p asc )
	       SELECT distinct n
	       FROM Q qq;


END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 241 - Task 1 - Python Implementation

Again, a triple nested loop implementation. Note however, how it is required to transform the values into integers.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    diff = int(argv[ 0 ])
    nums = list( map( int, argv[ 1: ] ) )
    counter = 0

    for i in range( 0, len( nums ) ):
        for j in range( i + 1, len( nums ) ):
            for k in range( j + 1, len( nums ) ):
               if ( ( nums[ k ] - nums[ j ] ) == ( nums[ j ] - nums[ i ] ) and ( nums[ k ] - nums[ j ] ) == diff ):
                   counter += 1

    print( counter )
    return counter



# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )



```
<br/>
<br/>



<a name="task2python"></a>
## PWC 241 - Task 2 - Python Implementation

The same implementation used for the other Raku and Perl implementation: a dictionary stores the values keyed by the counting of the prime factors.


<br/>
<br/>
```python
import sys

def get_primes( n ):
    primes = []
    for i in range(2, n):
        while n % i == 0:
            primes.append( i )
            n /= i

        if n == 1:
            break

    return primes

# task implementation
def main( argv ):
    sorted_dict = {}
    for current in argv:
        counter = len( get_primes( int( current ) ) )
        if not counter in sorted_dict:
            sorted_dict[ counter ] = []

        sorted_dict[ counter ].append( current )

    for key in sorted( sorted_dict ):
        for v in sorted_dict[ key ]:
            print( v )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )



```
<br/>
<br/>
