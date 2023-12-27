---
layout: post
title:  "Perl Weekly Challenge 249: the last one (for 2023)!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 249: the last one (for 2023)!

This post presents my solutions to the [Perl Weekly Challenge 249](https://perlweeklychallenge.org/blog/perl-weekly-challenge-249/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 249 - Task 1 - Raku](#task1)
- [PWC 249 - Task 2 - Raku](#task2)
- [PWC 249 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 249 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 249 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 249 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 249 - Task 1 in Python](#task1python)
- [PWC 249 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 249 - Task 1 - Raku Implementation

Given a list of integers, builds pairs of same integers.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems %% 2 && @nums.grep( * ~~ Int ).elems == @nums.elems } ) {

    my %pair-elements;
    %pair-elements{ $_ }++ for @nums;
    my @pairs;
    for %pair-elements.keys.sort {
		my $how-many-pairs = %pair-elements{ $_ } / 2;
		next if $how-many-pairs < 1;
		@pairs.push: [ $_, $_ ] while ( $how-many-pairs-- > 1 );
    }

    @pairs.join( ", " ).say;
}

```
<br/>
<br/>


The idea is that `@pairs` will contain the list of pairs to print out, and I first *classify* the numbers to count how much occurencies are there. If the occurrencies are so that another pairs can be built, I push it to the list of pairs.

<a name="task2"></a>
## PWC 249 - Task 2 - Raku Implementation

Given a string made of `D` or `I` (repeated), find a permutations that respects certain requirements given that the permutation index drives the letters to the requirements.

<br/>
<br/>
```raku
sub MAIN( $string  ) {

    my @chars = $string.comb;
    for ( 0 .. @chars.elems ).permutations -> $perm {
		my $ok = True;
		for 0 ..^ $perm.elems - 1 -> $i {
		    $ok = False and next if @chars[ $i ] ~~ 'D' && $perm[ $i ] < $perm[ $i + 1 ];
		    $ok = False and next if @chars[ $i ] ~~ 'I' && $perm[ $i ] > $perm[ $i + 1 ];
			last if ! $ok;
		}

	$perm.join(", ").say and last if $ok;
    }
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 249 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, I classify first the digits, then return all the pairs.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc249.task1_plperl( int[] )
RETURNS SETOF int[]
AS $CODE$

   my ( $nums ) = @_;

   my $classification = {};

   $classification->{ $_ }++ for ( $nums->@* );

   for ( sort keys $classification->%* ) {
       my $how_many_pairs = $classification->{ $_ } / 2;
       next if $how_many_pairs < 1;

       return_next( [ $_, $_ ] ) while ( $how_many_pairs-- >= 1 );

   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 249 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, I need to run it as untrusted since I use an external module for permutations.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc249.task2_plperl( text )
RETURNS int[]
AS $CODE$
   my ( $string ) = @_;

  use Algorithm::Combinatorics qw(permutations);

  my @chars = split //, $string;
  my @nums = 0 .. $#chars;

  my $iter = permutations( \ @nums );
  while (my $perm = $iter->next) {
  	my $ok = 1;
  	for my $i ( 0 .. scalar( $perm->@* ) - 1 ) {
	    $ok = 0 if ( $chars[ $i ] eq 'D' && $perm->[ $i ] < $perm->[ $i + 1 ] );
	    $ok = 0 if ( $chars[ $i ] eq 'I' && $perm->[ $i ] > $perm->[ $i + 1 ] );
	    last if ! $ok;
	}

	return $perm if $ok;
  }

  return undef;

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 249 - Task 1 - PL/PgSQL Implementation

I use a temporary table to store the counting of every digit, and then walk the table to get all the possible pairs to return.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc249.task1_plpgsql( n int[] )
RETURNS SETOF int[]
AS $CODE$
DECLARE
	r record;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS classification( v int, c int DEFAULT 1);
	TRUNCATE classification;


	WITH nums AS ( SELECT v::int
	    	      FROM unnest( n ) v )
	, counting AS ( SELECT v, count( v ) as c
	  	        FROM nums
			GROUP BY v )
	INSERT INTO classification( v, c )
	SELECT v, c
	FROM counting;

	FOR r IN SELECT * FROM classification WHERE c >= 2 ORDER BY v LOOP
	    WHILE r.c >= 1 LOOP
	    	  RETURN NEXT array[ r.v, r.v ]::int[];
		  r.c := r.c - 2;
	    END LOOP;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 249 - Task 2 - PL/PgSQL Implementation

I cheat on this: since doing permutations in SQL is too hard, I delegated to the Perl implementation this task!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc249.task2_plpgsql( s text )
RETURNS int[]
AS $CODE$
   SELECT pwc249.task2_plperl( s );
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 249 - Task 1 - Python Implementation

Quite simple implementation, based on the same approach used in Perl.
Note that I do not convert digits into numbers, so that I can quickly add them as keys in the dictionary.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    classification = {}
    for i in argv:
        if not i in classification:
            classification[ i ] = 0
        classification[ i ] += 1

    pairs = []
    for k in classification:
        while classification[ k ] >= 2:
            pairs.append( [ k, k ] )
            classification[ k ] -= 2

    for p in pairs:
        print( ", ".join( p ) )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )



```
<br/>
<br/>



<a name="task2python"></a>
## PWC 249 - Task 2 - Python Implementation

Similar approach to that of PL/Perl.

<br/>
<br/>
```python
import sys
from itertools import permutations

# task implementation
def main( argv ):
    nums = range( 0, len( argv[ 0 ] ) )


    for perm in permutations( nums ):
        ok = True
        for i in range( 0, len( perm ) - 1 ):
            if argv[ 0 ][ i ] == 'D' and perm[ i ] > perm[ i + 1 ]:
                ok = False
                break
            elif argv[ 0 ][ i ] == 'I' and perm[ i ] < perm[ i + 1 ]:
                ok = False
                break

        if ok:
            print( ",".join( map( str, perm ) ) )
            return


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )


```
<br/>
<br/>
