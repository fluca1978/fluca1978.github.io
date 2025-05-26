---
layout: post
title:  "Perl Weekly Challenge 323: increment and decrement"
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

# Perl Weekly Challenge 323: increment and decrement

This post presents my solutions to the [Perl Weekly Challenge 323](https://perlweeklychallenge.org/blog/perl-weekly-challenge-323/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 323 - Task 1 - Raku](#task1)
- [PWC 323 - Task 2 - Raku](#task2)
- [PWC 323 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 323 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 323 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 323 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)
- [PWC 323 - Task 1 in PL/Java](#task1pljava)
- [PWC 323 - Task 2 in PL/Java](#task2pljava)
- [PWC 323 - Task 1 in Python](#task1python)
- [PWC 323 - Task 2 in Python](#task2python)


The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 323 - Task 1 - Raku Implementation

The first task was about to compute the final value of a set of inline increments `++` and decrements `--`.

<br/>
<br/>
```raku
sub MAIN( *@operations ) {
    my $value = 0;
    for @operations {
		$value++ and next if ( $_ ~~ / '++' / );
		$value-- and next if ( $_ ~~ / '--' / );
    }

    $value.say;
}

```
<br/>
<br/>

If the current input param contains the `++` string I increment the current `$value`, otherwise I decrement it.


<a name="task2"></a>
## PWC 323 - Task 2 - Raku Implementation

The second task was about getting an income value, and a list of taxes that are grouped by two: the first value is the *up-to* value the second value, the second value applie to.


<br/>
<br/>
```raku
sub MAIN( Int $income, *@taxes where { @taxes.grep( * ~~ Int ).elems == @taxes.elems } ) {
    my $value = 0;
    my $last = 0;

    say @taxes;
    for @taxes -> $up-to, $pct {
		next if $last > $income;
		$value += ( min( $up-to, $income ) - $last )  * $pct / 100;
		$last = $up-to;
    }

    $value.say;

}

```
<br/>
<br/>

I store into `$last` the last seen up-to value, so that I can compute how much income remains to count for the next iteration.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 323 - Task 1 - PL/Perl Implementation

Same as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc323.task1_plperl( text[] )
RETURNS int
AS $CODE$

   my ( $operations ) = @_;
   my $value = 0;

   for ( $operations->@* ) {
       $value++ and next if ( $_ =~ / \+ \+ /x );
       $value-- and next if ( $_ =~ / \- \- /x );
   }

   return $value;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 323 - Task 2 - PL/Perl Implementation


Same as in Raku.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc323.task2_plperl( int, int[] )
RETURNS float
AS $CODE$

   my ( $income, $taxes ) = @_;
   my $value = 0;
   my $last = 0;


   my $min = sub { $_[ 0 ] > $_[ 1 ] ? $_[ 1 ] : $_[ 0 ] };

   while ( $taxes->@* ) {
   	 my $up_to = shift $taxes->@*;
	 my $pct   = shift $taxes->@*;

	 next if ( $last > $income );
	 $value += ( $min->( $up_to, $income ) - $last ) * $pct / 100;
	 $last = $up_to;
   }

   return $value;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 323 - Task 1 - PL/PgSQL Implementation

Similar to the PL/Perl implementation, I use a regular expression to see what operation does match.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc323.task1_plpgsql( operations text[] )
RETURNS int
AS $CODE$
DECLARE
	v int := 0;
	current text;
BEGIN

	FOREACH current IN ARRAY operations LOOP
		IF regexp_like( current, '\+\+' ) THEN
		   v := v + 1;
		ELSIF regexp_like( current, '\-\-' ) THEN
		   v := v - 1;
		END IF;
	END LOOP;

	RETURN v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 323 - Task 2 - PL/PgSQL Implementation

Similar to the PL/Perl, I iterate on all the taxes skipping a value very couple.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc323.task2_plpgsql( income int, taxes int[] )
RETURNS float
AS $CODE$
DECLARE
	v float := 0;
	previous int := 0;
	i int;
	up_to int;
	pct float;
BEGIN

	FOR i IN 1 .. array_length( taxes, 1 ) LOOP
	    IF mod( i, 2 ) = 0 THEN
	       CONTINUE;
	    END IF;

	    pct := taxes[ i + 1 ];
	    up_to := taxes[ i ];

	    IF previous > income THEN
	       CONTINUE;
	    END IF;

	    IF up_to < income THEN
	       v := v + ( up_to - previous ) * pct / 100;
	    ELSE
	      v := v + ( income - previous ) * pct / 100;
	    END IF;

            previous := up_to;

	END LOOP;

	RETURN v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


# Java Implementations

<a name="task1pljava"></a>
## PWC 323 - Task 1 - PostgreSQL PL/Java Implementation

Very simple implementation using `String.contains` instead of a regular expression.


<br/>
<br/>
```java
    public static final int task1_pljava( String[] operations ) throws SQLException {
	   logger.log( Level.INFO, "Entering pwc323.task1_pljava" );

	   int value = 0;

	   for ( String op : operations ) {
	       if ( op.contains( "++" ) )
	   	value++;
	       else if ( op.contains( "--" ) )
	   	value--;
	   }

	   return value;
    }
```
<br/>
<br/>



<a name="task2pljava"></a>
## PWC 323 - Task 2 - PostgreSQL PL/Java Implementation

An approach that iterates over the taxes, as in PL/Perl.

<br/>
<br/>
```java
    public static final float task2_pljava( int income, int[] taxes ) throws SQLException {
		logger.log( Level.INFO, "Entering pwc323.task2_pljava" );

		int last = 0;
		float value = 0;

		for ( int i = 0; i < taxes.length - 1; i++, i++ ) {
		    float pct = taxes[ i + 1 ];
		    int up_to = taxes[ i ];

		    if ( last > up_to )
			continue;

		    value += Math.min( up_to, income ) * pct / 100;
		    last = up_to;
		}

		return value;

    }
```
<br/>
<br/>


# Python Implementations

<a name="task1python"></a>
## PWC 323 - Task 1 - Python Implementation

Same approach as in other implementations.

<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_1( args ):
    value = 0

    for operation in args:
        if '++' in operation:
            value += 1
        elif '--' in operation:
            value -= 1

    return value


# invoke the main without the command itself
if __name__ == '__main__':
    print( task_1( sys.argv[ 1: ] ) )

```
<br/>
<br/>



<a name="task2python"></a>
## PWC 323 - Task 2 - Python Implementation

Same approach as in other implementations.


<br/>
<br/>
```python
import sys

# task implementation
# the return value will be printed
def task_2( args ):
    income = int( args[ 0 ] )
    taxes = list( map( int, args[ 1: ] ) )
    value = 0
    last = 0

    for i in range( 0, len( taxes ) - 1 ) :
        if i % 2 == 0:
            continue


        up_to = taxes[ i ]
        pct   = taxes[ i + 1 ]

        if last > up_to:
            continue

        value += min( up_to, income ) * pct / 100

    return value

# invoke the main without the command itself
if __name__ == '__main__':
    print( task_2( sys.argv[ 1: ] ) )

```
<br/>
<br/>
