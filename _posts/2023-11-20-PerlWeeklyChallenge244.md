---
layout: post
title:  "Perl Weekly Challenge 244: grep and filter everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 244: grep and filter everywhere!

This post presents my solutions to the [Perl Weekly Challenge 244](https://perlweeklychallenge.org/blog/perl-weekly-challenge-244/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 244 - Task 1 - Raku](#task1)
- [PWC 244 - Task 2 - Raku](#task2)
- [PWC 244 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 244 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 244 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 244 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 244 - Task 1 in Python](#task1python)
- [PWC 244 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 244 - Task 1 - Raku Implementation

The first task was about producing an array of integers where each element is the counting of how many integers are greater than the given input valu within an array of integers.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {
    my @result;
    @result[ $_ ] = @nums.grep( * > @nums[ $_ ] ).elems  for 0 ..^ @nums.elems;
    @result.say;
}

```
<br/>
<br/>



<a name="task2"></a>
## PWC 244 - Task 2 - Raku Implementation

The second task was about computing the *power* of a group of heroes, each one with a given strenght in an array of integers.
There was the need to comput all the possible combinations of heroes.


<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.elems == @nums.grep( * ~~ Int ).elems } ) {

    my $power = 0;
    for @nums.combinations {
		next if ! $_;
		$power +=  $_.min * ( $_.max  ** 2 );
    }

    $power.say;
}

```
<br/>
<br/>


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 244 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc244.task1_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;

   for my $needle ( $nums->@* ) {
       return_next( scalar( grep( { $_ > $needle } $nums->@* ) ) );
   }

   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 244 - Task 2 - PL/Perl Implementation

I decide to use two libraries, `List::Util` and `Alghoritm::Combinatorics` to get functions that can help me get all the combinations and the needed min and max values out of an array.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc244.task2_plperl( int[] )
RETURNS int
AS $CODE$
   use Algorithm::Combinatorics qw/ combinations /;
   use List::Util qw/ min max /;

   my ( $nums ) = @_;
   my $power;

   $power = 0;

   for my $k ( 1 .. $nums->@* ) {
       my $combinations = combinations( \ $nums->@*, $k );
       while ( my $iter = $combinations->next ) {
       	     $power += min( $iter->@* ) * ( max( $iter->@* ) ** 2 );
       }
   }


   return $power;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

There is an interesting thing to note here: even if `$nums` is *like* an array reference, the `combinations` method does not accept it as an arrayref, and therefore I needed to transform it into an array and get back the explicit reference.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 244 - Task 1 - PL/PgSQL Implementation

It is easy enough to get the count of the elements greater than the given one, thanks to a single query.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc244.task1_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	v int;
	returning int;
BEGIN
	FOREACH v IN ARRAY nums LOOP
		SELECT count(*)
		INTO returning
		FROM unnest( nums ) n
		WHERE n > v;

		RETURN NEXT returning;
	END LOOP;

	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 244 - Task 2 - PL/PgSQL Implementation

I cheated here! Since obtaining all the combinations is too much work to be done in SQL, I delegated the PL/Perl to answer the task.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc244.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
   SELECT pwc244.task2_plperl( nums );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



# Python Implementations

<a name="task1python"></a>
## PWC 244 - Task 1 - Python Implementation

I use `filter` as an alternative to the Perl's `grep` function, then I append the single result into the `result` array and output it.

<br/>
<br/>
```python
import sys

# task implementation
def main( argv ):
    nums = list( map( int, argv ) )
    result = []

    for current in nums:
        result.append( len( list( filter( lambda x: x > current, nums ) ) ) )

    print( ", ".join( map( str, result ) ) )


# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )


```
<br/>
<br/>



<a name="task2python"></a>
## PWC 244 - Task 2 - Python Implementation

I use `itertoolsd.combinations` to get all the possible combinations, even if I need one unit more than the `len` of the array.

<br/>
<br/>
```python
import sys
from itertools import combinations;

# task implementation
def main( argv ):
    nums = list( map( int, argv ) )
    power = 0

    for size in range( 1, len( nums ) + 1 ):
        for current in combinations( nums, size ):
            power = power + ( max( current ) ** 2 ) * min( current )

    print( power )



# invoke the main without the command itself
if __name__ == '__main__':
    main( sys.argv[ 1: ] )

```
<br/>
<br/>
