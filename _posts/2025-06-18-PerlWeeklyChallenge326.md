---
layout: post
title:  "Perl Weekly Challenge 326: Happy Birthday Ma'!"
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

# Perl Weekly Challenge 326: Happy Birthday Ma'!

This post presents my solutions to the [Perl Weekly Challenge 326](https://perlweeklychallenge.org/blog/perl-weekly-challenge-326/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 326 - Task 1 - Raku](#task1)
- [PWC 326 - Task 2 - Raku](#task2)
- [PWC 326 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 326 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 326 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 326 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 326 - Task 1 in PL/Java](#task1pljava)
- [PWC 326 - Task 2 in PL/Java](#task2pljava)
- [PWC 326 - Task 1 in Python](#task1python)
- [PWC 326 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 326 - Task 1 - Raku Implementation

The first task was about to find out the *day of the year* of a given date, in the format `yyyy-mm-dd`.

<br/>
<br/>
```raku
sub MAIN( Str $date ) {
    my $day = Date.new: $date;
    say 1 + $day - Date.new: month => 1, day => 1, year => $day.year;
}

```
<br/>
<br/>

Given the string, it is simple enough to convert it into a `Date` object and then to compute the difference in days with the first date of the year.


<a name="task2"></a>
## PWC 326 - Task 2 - Raku Implementation

Given an array of integer, *compact* it so that the first item is repeated the number of times of the second item, and so on.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {
    my @result;
    for @nums -> $base, $times {
		@result.push: ( $base xx $times );
    }

    @result.flat.say;
}

```
<br/>
<br/>

Thanks to the `xx` list operator, it is very simple to create a list out of the `$base` knowing how many `$times` it has to be repeated.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 326 - Task 1 - PL/Perl Implementation

Even simpler than the Raku solution, since `DateTime` provides a specific method to get the `day_of_year`.
Note that there is the need to run it as `plperlu` because the usage of `DateTime` module.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc326.task1_plperl( text )
RETURNS int
AS $CODE$

   use DateTime;
   my ( $date ) = @_;

   if ( $date =~ / ^ (\d{4}) - (\d{2}) - (\d{2}) $ /x ) {

      my $day = DateTime->new( year  => $1,
                               month => $2,
   			    day   => $3 );

      return $day->day_of_year;
   }

   return 0;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 326 - Task 2 - PL/Perl Implementation

The idea is the same as in Raku implementation, but I use a `for` loop to provide the sublists.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc326.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$

   my ( $nums ) = @_;
   my @result;

   for my $index ( 0 .. $nums->@* - 2 ) {
       next unless ( $index % 2 == 0 );
      my ( $base, $times ) = $nums->@[ $index .. $index + 1 ];
      push @result, $base for ( 0 .. $times );

   }

   return [ @result ];

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 326 - Task 1 - PL/PgSQL Implementation

A single query can do the trick.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc326.task1_plpgsql( d date )
RETURNS int
AS $CODE$
   SELECT 1 +
          d -
	  ( extract( year from d ) || '-01-01' )::date;
$CODE$
LANGUAGE sql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 326 - Task 2 - PL/PgSQL Implementation

Similar to the PL/Perl implementation.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc326.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	i int;
	base int;
	times int;
BEGIN

	FOR i IN 1 .. array_length( nums, 1 ) - 1 LOOP
	    IF mod( i, 2 ) = 0 THEN
	       CONTINUE;
	    END IF;

	    base  := nums[ i ];
	    times := nums[ i + 1 ];


	    FOR j in 1 .. times LOOP
	    	RETURN NEXT base;
            END LOOP;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 326 - Task 1 - PostgreSQL PL/Java Implementation

Since I don't know enought about the "new" Java timing classes, I use the bovine approach of computing the milliseconds and converting them to days.

<br/>
<br/>
```java
    public static final int task1_pljava( Date day ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc326.task1_pljava" );

		Calendar end   = Calendar.getInstance();
		end.setTime( day );
		Calendar begin = (Calendar) end.clone();
		begin.set( Calendar.DAY_OF_YEAR, 1 );
		return (int) ( ( end.getTimeInMillis() - begin.getTimeInMillis() ) / ( 1000 * 60 * 60 * 24 ) );
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 326 - Task 2 - PostgreSQL PL/Java Implementation

An iterative approach.

<br/>
<br/>
```java
    public static final int[] task2_pljava( int[] nums ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc326.task2_pljava" );

		List<Integer> result = new LinkedList<Integer>();
		for( int i = 0; i < nums.length - 1; i++ ) {
		    if( i % 2 != 0 )
				continue;

		    int base  = nums[ i ];
		    int times = nums[ i + 1 ];

		    for ( int j = 0; j < times; j++ )
				result.add( base );
		}

		int r[] = new int[ result.size() ];
		for ( int i = 0; i < result.size(); i++ )
		    r[ i ] = result.get( i );

		return r;
    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 326 - Task 1 - Python Implementation

It is only a matter of computing the diff between dates.

<br/>
<br/>
```python
import sys
from datetime import datetime, date

# task implementation
# the return value will be printed
def task_1( args ):
    begin = datetime.strptime( args[ 0 ], "%Y-%m-%d").date()
    end   = date( begin.year, 1, 1 )
    return abs( ( end - begin ).days )


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 326 - Task 2 - Python Implementation

Similar to the PL/Java implementation.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    nums = list( map( int, args ) )

    result = []

    for i in range( 0, len( nums ) - 1 ):
        if i % 2 != 0 :
            continue

        base  = nums[ i ]
        times = nums[ i + 1 ]

        for x in range(0, times ):
            result.append( base )

    return result


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
