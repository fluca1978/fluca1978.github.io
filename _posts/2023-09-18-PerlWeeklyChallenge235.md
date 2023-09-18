---
layout: post
title:  "Perl Weekly Challenge 235: integer arrays"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 235: integer arrays

This post presents my solutions to the [Perl Weekly Challenge 235](https://perlweeklychallenge.org/blog/perl-weekly-challenge-235/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 235 - Task 1 - Raku](#task1)
- [PWC 235 - Task 2 - Raku](#task2)
- [PWC 235 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 235 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 235 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 235 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 235 - Task 1 - Raku Implementation

The first task was about discovering if a given input array of integers is naturally sorted removing only one item.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {

    for 0 ..^ @nums.elems {
		my @current;
		@current.push: | @nums[ 0 .. $_ - 1 ] if $_ != 0;
		@current.push: | @nums[ $_ + 1 ..^ @nums.elems ];

		'true'.say and exit if @current ~~ @current.sort;
    }

    'false'.say;
}
```
<br/>
<br/>


I build at every step an array made by a slice of the original array, and compare it with its sorted version. If both the array are the same, the result is `true`.


<a name="task2"></a>
## PWC 235 - Task 2 - Raku Implementation

The second task was about duplicating every `0` found in a given array of integers, without exceeding the array size, that means trailing the extra digits.

<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~ Int ).elems == @nums.elems } ) {

    my @result;
    for 0 ..^ @nums.elems {
		# add the leading zero
		@result.push: 0  if @nums[ $_ ] == 0;
		# add the number, even if it is a zero, duplicated
		# from the row above
		@result.push: @nums[ $_ ];
    }

    @result[ 0 ..^ @nums.elems ].join( ',' ).say;
}

```
<br/>
<br/>

I add every digit to the `@result` array, and add it twice if it is a zero. In the end, I output only the slice of the array.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 235 - Task 1 - PL/Perl Implementation

A bit by bit similar solution to the Raku one.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc235.task1_plperl( int[] )
RETURNS boolean
AS $CODE$
   my ( $nums ) = @_;

   for ( 0 .. $nums->@* - 1 ) {
       my @current;
       push @current, $nums->@[ 0 .. $_ - 1 ] if ( $_ != 0 );
       push @current, $nums->@[ $_ + 1 .. $nums->@* - 1 ];

       return 1 if ( join( ',', @current ) eq join( ',', sort @current ) );
   }

   return 0;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


This time, I compare the *stringified* version of the array and of the sorted one to see it if matches.


<a name="task2plperl"></a>
## PWC 235 - Task 2 - PL/Perl Implementation

This time I return every digit when I encounter it, stopping when the max size is found.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc235.task2_plperl( int[] )
RETURNS SETOF int
AS $CODE$
   my ( $nums ) = @_;
   my $remaining = $nums->@* - 1;

   for ( $nums->@* ) {

       return_next( $_ );
       $remaining--;
       return undef if $remaining <= 0;

       return_next( 0 ) if ( $_ == 0 );
       $remaining--;
       return undef if $remaining <= 0;

   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 235 - Task 1 - PL/PgSQL Implementation

A quite verbose solution, but nothing better came into my mind.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc235.task1_plpgsql( nums int[] )
RETURNS boolean
AS $CODE$
DECLARE
	current_array int[];
	string_array  text;
	string_sorted text;
BEGIN

        SELECT string_agg( x::text, ',' )
	INTO string_array
	FROM ( SELECT unnest( nums ) y ORDER BY 1 ) x
	ORDER BY 1;


	FOR i IN 1 .. array_length( nums, 1 ) LOOP

	    current_array := '{}';

	    IF i <> 1 THEN
	       	current_array := current_array || nums[ 1 : ( i - 1 ) ];
	    END IF;

	    current_array := current_array || nums[ ( i + 1 ) : ];

	    SELECT string_agg( x::text, ',' )
	    INTO string_sorted
	    FROM ( SELECT y::text FROM unnest( current_array ) y ORDER BY 1 ) x
	    ;


	    SELECT string_agg( x::text, ',' )
	    INTO string_array
	    FROM ( SELECT y::text FROM unnest( current_array ) y ) x
	    ;

	    IF string_sorted = string_array THEN
	       RETURN true;
	    END IF;

	END LOOP;

	RETURN false;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The idea is to slice the original array into one that has all elements except one. Then I build a string representation of the built array and of the sorted one, and if the strings are the same, I can stop.


<a name="task2plpgsql"></a>
## PWC 235 - Task 2 - PL/PgSQL Implementation

Simple solution using the same PL/Perl approach: return every digit, return a zero twice, stop as soon as the max size is reached.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc235.task2_plpgsql( nums int[] )
RETURNS SETOF int
AS $CODE$
DECLARE
	current_element int;
	remaining int;
BEGIN
	remaining := array_length( nums, 1 );

	FOREACH current_element IN ARRAY nums LOOP
		RETURN NEXT current_element;
		remaining := remaining - 1;

		IF remaining = 0 THEN
		   RETURN;
		END IF;

		IF current_element = 0 THEN
		   RETURN NEXT 0;
		   remaining := remaining - 1;
		END IF;

		IF remaining = 0 THEN
		   RETURN;
		END IF;

	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>
