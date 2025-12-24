---
layout: post
title:  "Perl Weekly Challenge 353: waiting for Santa..."
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

# Perl Weekly Challenge 353: waiting for Santa...

This post presents my solutions to the [Perl Weekly Challenge 353](https://perlweeklychallenge.org/blog/perl-weekly-challenge-353/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 353 - Task 1 - Raku](#task1)
- [PWC 353 - Task 2 - Raku](#task2)
- [PWC 353 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 353 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 353 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 353 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 353 - Task 1 in PL/Java](#task1pljava)
- [PWC 353 - Task 2 in PL/Java](#task2pljava)
- [PWC 353 - Task 1 in Python](#task1python)
- [PWC 353 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 353 - Task 1 - Raku Implementation

The first task was about finding the sentence, given a list in input, with the max number of words.
Easy enough to do in a single line!

<br/>
<br/>
```raku
sub MAIN( *@sentences ) {
    @sentences.map( *.split( /\s+/ ).elems ).max.say;
}

```
<br/>
<br/>

Split every sentence into words, map to the number of words, extract the max value and print it.


<a name="task2"></a>
## PWC 353 - Task 2 - Raku Implementation

The second task was about correlating three arrays to evaluate to `true` or `felase` depending on the value of every entry.

<br/>
<br/>
```raku
sub MAIN(  :@codes,  :@names,  :@status ) {

    my @valid;

    for 0 ..^ @codes.elems  {
		@valid[ $_ ] = False and next unless ( @codes[ $_ ] );
		@valid[ $_ ] = False and next unless ( @codes[ $_ ] ~~ / ^ <[a..zA..Z0..9_]>+ $ / );
		@valid[ $_ ] = False and next unless ( @names[ $_ ] ~~ / electronics|grocery|pharmacy|restaurant / );
		@valid[ $_ ] = False and next unless ( @status[ $_ ].so );
		@valid[ $_ ] = True;
    }

    @valid.say;

}

```
<br/>
<br/>


# PL/Perl Implementations

The same as in Raku, only verbose.

<a name="task1plperl"></a>
## PWC 353 - Task 1 - PL/Perl Implementation

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc353.task1_plperl( text[] )
RETURNS int
AS $CODE$

   my ( $sentences ) = @_;

   my @w = $sentences->@*;
   return ( sort { $b <=> $a }
	    map { scalar split /\s+/, $_   } $sentences->@* )[ 0 ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

I use the trick of sorting the array and get the first element in order to get the max value.


<a name="task2plperl"></a>
## PWC 353 - Task 2 - PL/Perl Implementation

Similar to the Raku approach.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc353.task2_plperl( text[], text[], text[] )
RETURNS SETOF text
AS $CODE$

   my ( $codes, $names, $status ) = @_;

   for ( 0 .. $codes->@* - 1 ) {
       my $valid = 1;

       $valid = 0 unless ( $codes->@[ $_ ] =~ / ^ [a-zA-Z0-9_]+ $ /x );
       $valid = 0 unless ( $status->@[ $_ ] =~ /true/i );
       $valid = 0 unless ( $names->@[ $_ ] =~ /electronics|grocery|pharmacy|restaurant/i );
       return_next( $valid ? 'true' : 'false' );
   }

   return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 353 - Task 1 - PL/PgSQL Implementation

A single query does suffice!

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc353.task1_plpgsql( s text[] )
RETURNS int
AS $CODE$
   SELECT max( regexp_count( x, '\w+' ) )
   FROM   unnest( s ) x;

$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 353 - Task 2 - PL/PgSQL Implementation


Here I cheated, and use PL/Perl to do the trick.
<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc353.task2_plpgsql( c text[], n text[], s text[] )
RETURNS SETOF text
AS $CODE$
   SELECT pwc353.task2_plperl( c, n, s );

$CODE$
LANGUAGE sql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 353 - Task 1 - PostgreSQL PL/Java Implementation

Quite simple with an iteration.

<br/>
<br/>
```java
    public static final int task1_pljava( String[] sentences ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc353.task1_pljava" );

		int max = 0;
		for ( String s : sentences ) {
		    int current = s.split( "\\s+" ).length;
		    if ( current > max )
				max = current;
		}

		return max;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 353 - Task 2 - PostgreSQL PL/Java Implementation

Simple enough to evaluate in a loop.

<br/>
<br/>
```java
    public static final Boolean[] task2_pljava( String codes[],
						String names[],
						String status[] )
	throws SQLException {
		logger.log( Level.INFO, "Entering pwc353.task2_pljava" );

		Boolean result[] = new Boolean[ codes.length ];

		Pattern regex = Pattern.compile( "^[a-z0-9_]+$",
						 Pattern.CASE_INSENSITIVE );


		for ( int i = 0; i < codes.length; i++ ) {
		    	Matcher engine = regex.matcher( codes[ i ] );

			if ( engine.find()
			     && ( names[ i ].equalsIgnoreCase( "electronics" )
				  || names[ i ].equalsIgnoreCase( "grocery" )
				  || names[ i ].equalsIgnoreCase( "pharmacy" )
				  || names[ i ].equalsIgnoreCase( "restaurant" ) )
			     && status[ i ].equalsIgnoreCase( "true" ) )
			    result[ i ] = true;
			else
			    result[ i ] = false;
		}

		return result;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 353 - Task 1 - Python Implementation

A single line suffice.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_1( args ):
    regex = re.compile( r'\s+' )
    return max( list( map( lambda x: len( regex.split( x ) ), args ) ) )



# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 353 - Task 2 - Python Implementation

Same PL/Java implementation.

<br/>
<br/>
```python
import sys
import re

# task implementation
# the return value will be printed
def task_2( args ):
    result  = []
    regex   = re.compile( r'\s+' )
    codes   = list( regex.split( args[0] ) )
    names   = list( regex.split( args[1] ) )
    status  = list( regex.split( args[2] ) )


    r1 = re.compile( r'^[a-zA-Z0-9_]+$' )
    r2 = re.compile( r'^electronics|grocery|pharmacy|restaurant$' )

    for i in range( 0, len( codes ) ):
        if r1.match( codes[ i ] ) and r2.match( names[ i ] ) and status[ i ] == "true":
            result.append( "true" )
        else:
            result.append( "false" )

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
