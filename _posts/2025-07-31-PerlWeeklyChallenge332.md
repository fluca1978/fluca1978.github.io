---
layout: post
title:  "Perl Weekly Challenge 332: quick and easy"
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

# Perl Weekly Challenge 332: quick and easy

This post presents my solutions to the [Perl Weekly Challenge 332](https://perlweeklychallenge.org/blog/perl-weekly-challenge-332/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 332 - Task 1 - Raku](#task1)
- [PWC 332 - Task 2 - Raku](#task2)
- [PWC 332 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 332 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 332 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 332 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 332 - Task 1 in PL/Java](#task1pljava)
- [PWC 332 - Task 2 in PL/Java](#task2pljava)
- [PWC 332 - Task 1 in Python](#task1python)
- [PWC 332 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 332 - Task 1 - Raku Implementation

The first task was about traducing an incoming date in the format `YYYY-MM-DD` into a binary numeric representation.

<br/>
<br/>
```raku
sub MAIN( Str $date where { $date ~~ / ^ \d ** 4 '-' \d ** 2 '-' \d ** 2 $/ } ) {
    my @bins;
    for $date.split( '-' ) {
		@bins.push: $_.Int.base( 2 );
    }

    @bins.join( '-' ).say;
}

```
<br/>
<br/>

Quite simple: I `split` the string into its numerical parts, convert every number into a binary value with `base` and rejoin the result.


<a name="task2"></a>
## PWC 332 - Task 2 - Raku Implementation

The second task was about to find out if a given string has only unique letters.

<br/>
<br/>
```raku
sub MAIN( Str $string ) {
    my $bag = Bag.new: $string.comb;
    'false'.say and exit if ( $bag.values.grep( * > 1 ) );
    'true'.say;
}

```
<br/>
<br/>


A `Bag` does the trick, since it can be indexed by the single letter and it counts the occurrencies. Hence, if the values are not set to `1`, at least one letter is repeated.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 332 - Task 1 - PL/Perl Implementation

Similar to Raku, split the string, convert using `sprintf` and rejoin the resulting values.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc332.task1_plperl( text )
RETURNS text
AS $CODE$

   my ( $date ) = @_;
   my @bins;

   for ( split /[-]/, $date ) {
       push @bins, sprintf( '%b', $_ );
   }

   return join( '-', @bins );

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 332 - Task 2 - PL/Perl Implementation

Use an hash to implement the same bag behavior.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc332.task2_plperl( text )
RETURNS boolean
AS $CODE$

   my ( $string ) = @_;
   my $bag = {};

   for ( split //, $string ) {
       $bag->{ $_ }++;
   }

   return 0 if ( grep { $_ > 1 } values $bag->%* );
   return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 332 - Task 1 - PL/PgSQL Implementation

A single query can do the trick!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc332.task1_plpgsql( d date )
RETURNS text
AS $CODE$
   SELECT string_agg( v::int::bit( 8 )::text, '-' )
   FROM   regexp_split_to_table( d::text, '-' ) v;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 332 - Task 2 - PL/PgSQL Implementation

Again, a single query can do the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc332.task2_plpgsql( t text )
RETURNS boolean
AS $CODE$
   select NOT exists(
   	  select v
	  from regexp_split_to_table( t, '' ) v
	  group by v
	  having count(*) > 1 );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>

I split to a table the text, and search for repreated values.

# Java Implementations

<a name="task1pljava"></a>
## PWC 332 - Task 1 - PostgreSQL PL/Java Implementation

I misnumbered the tasks!
This is the second task.


<br/>
<br/>
```java
    public static final boolean task1_pljava( String text ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc332.task1_pljava" );

		Map<String, Integer> bag = new HashMap<String, Integer>();
		boolean ok = true;

		for ( String c : text.split( "" ) ) {
		    bag.putIfAbsent( c, 0 );
		    bag.put( c, bag.get( c ) + 1 );
		    if ( bag.get( c ) > 1 )
				ok = false;
		}

		return ok;

    }
```
<br/>
<br/>

As in PL/Perl, use a `Map` to implement a bag.


<a name="task2pljava"></a>
## PWC 332 - Task 2 - PostgreSQL PL/Java Implementation

This is the first task!


<br/>
<br/>
```java
    public static final String task2_pljava( String date ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc332.task2_pljava" );

		String result = "";
		int dashes = 2;
		for ( String current : date.split( "-" ) ) {
		    result += Integer.toBinaryString( Integer.parseInt( current ) );
		    if ( dashes > 0 ) {
				result += "-";
				dashes--;
		    }
		}

		return result;
    }
```
<br/>
<br/>

I use `toBinaryString` against the part of the string converted in `Integer`.


# Python Implementations

<a name="task1python"></a>
## PWC 332 - Task 1 - Python Implementation

Similar to the other implementations

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    bins = []
    for c in re.split( r'[-]', args[ 0 ] ) :
        bins.append( "{0:b}".format( int( c ) ) )

    return '-'.join( bins )

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>

I use a regular expression to split the string and `format` to convert the string value from an integer into a string of bits.

<a name="task2python"></a>
## PWC 332 - Task 2 - Python Implementation

Implement a bag behavior with a dictionary.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    bag = {}
    for c in args[ 0 ]:
        if not c in bag:
            bag[ c ] = 0
        bag[ c ] = bag[ c ] + 1

    for x in bag:
        if bag[ x ] > 1:
            return False

    return True


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
