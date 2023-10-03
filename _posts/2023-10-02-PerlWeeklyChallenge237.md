---
layout: post
title:  "Perl Weekly Challenge 237: in the need for caffeine!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 237: in the need for caffeine!

This post presents my solutions to the [Perl Weekly Challenge 237](https://perlweeklychallenge.org/blog/perl-weekly-challenge-237/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 237 - Task 1 - Raku](#task1)
- [PWC 237 - Task 2 - Raku](#task2)
- [PWC 237 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 237 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 237 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 237 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 237 - Task 1 - Raku Implementation

The first task was about finding out a date, writing it out in a verbose way, given the year, the month and the day of week and the week in the month.

<br/>
<br/>
```raku
sub MAIN( Int :$y, Int :$m, Int :$d, Int :$w ) {
    my $date = Date.new( year => $y,
			 month => $m,
			 day => 1 );

    my %ord = 1 => 'first', 2 => 'second', 3 => 'third';
    %ord{ $_ } = $_ ~ 'th' for ( 4 .. 31 );

    my %names = 1 => 'Monday',
		2 => 'Tuesday',
		3 => 'Wednsday',
		4 => 'Thursday',
		5 => 'Friday',
		6 => 'Saturday',
		7 => 'Sunday';

    my $current-week = 0;
    while ( $date.month == $m && $date.year == $y ) {

		$current-week++ if ( $date.day-of-week == 1 );
		if ( $date.day-of-week == $d && $current-week == $w ) {
		    "The { %ord{ $current-week } } { %names{ $date.day-of-week } } of month $m in year $y is { $date.day }".say;
		    exit;
		}
		$date++;
    }

    'Not found'.say;
}

```
<br/>
<br/>


I'm sure there is a smarter way, but today it does not come into my mind!

The idea is to start at the very first day of the month, and increase by one day trying to search if the week number becomes equal to the requested one, and if the day of the week is the same of the neeeded one. In such case, I write out the day and exit.


<a name="task2"></a>
## PWC 237 - Task 2 - Raku Implementation

Find out, in all permutations of an array, if the given permutation has a *greatest order* of the original one, that is which is the max element of the given permutation so that up to such index cells of the permutations are greater than those of the original array.


<br/>
<br/>
```raku
sub MAIN( *@nums where { @nums.grep( * ~~Int ).elems == @nums.elems } ) {

    my %permutations;

    for @nums.permutations -> $current-permutation {
		for 0 ..^ @nums.elems {
			if ( @nums[ $_ ] > $current-permutation[ $_ ] ) {
				%permutations.{ $_ }.push: $current-permutation;
				last;
            }
	    }
    }

    my $greatest = %permutations.keys.max;
    my $permutations = %permutations{ $greatest }.elems;
    "Greatest $greatest with $permutations possible permutations".say;
    %permutations{ $greatest }[ 0 ].join( ',' ).say;
}

```
<br/>
<br/>

I iterate on every permutation, then on every element of the obtained array and stop at the first index when the condition is not met. At that time, I push the permutation to the hash of the available `%permutations` keyed by the index itself. Then I extract the max value of keys and produce an output with an example of permutation.

# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 237 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, but use `DateTime` to obtain information about the day.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc237.task1_plperl( int, int, int, int )
RETURNS text
AS $CODE$
   my ( $year, $month, $weekday, $week ) = @_;
   my $current_week = 0;

   my $ord = { 1 => 'first', 2 => 'second', 3 => 'third' };
   $ord->{ $_ } = $_ . 'th' for ( 4 .. 31 );



   use DateTime;
   my $date = DateTime->new( year => $year, month => $month, day => 1 );

   while ( $date->year == $year && $date->month == $month ) {
   	 $current_week++ if ( $date->day_of_week == 1 );
	 if ( $current_week == $week && $date->day_of_week == $weekday ) {
	    # found
	    return sprintf 'The %s %s of month %s in %s is %d',
	    	   	   $ord->{ $current_week },
			   $date->day_abbr,
			   $date->month_abbr,
			   $date->year,
			   $date->day;
	 }

	 $date->add( days => 1 );
   }

   return 'Date not found';

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 237 - Task 2 - PL/Perl Implementation

Here I use `List::Permutor` to iterate on every permutation, but the inner alghoritm is the same as in the Raku implementation.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc237.task2_plperl( int[] )
RETURNS int
AS $CODE$
   my ( $nums ) = @_;

   my $permutations = {};

   use List::Permutor;
   my $engine = List::Permutor->new( $nums->@* );
   while ( my @current_permutation = $engine->next ) {
   	 for ( 0 .. $nums->@* ) {
	     if ( $nums->[ $_ ] > $current_permutation[ $_ ] ) {
	     	# stop here
			push $permutations->{ $_ }->@*, $current_permutation;
			last;
	     }
	 }
   }




   #seek the max key
   return ( sort keys $permutations->%* )[ -1 ];


$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 237 - Task 1 - PL/PgSQL Implementation

here I use a `date` type to begin at the very first day of the month, and increment by one day at the time.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc237.task1_plpgsql( y int, m int, d int, w int )
RETURNS text
AS $CODE$
DECLARE
	current_date date;
	current_week int := 0;
	current_result text;
	current_temp text;
BEGIN
	SELECT make_date( y, m, 1 )
	INTO current_date;


	CREATE TEMPORARY TABLE IF NOT EXISTS ord( o int, t text );
	TRUNCATE ord;
	INSERT INTO ord
	VALUES( 1, 'first' ), ( 2, 'second' ), (3, 'third' );
	FOR i IN 4 .. 31 LOOP
	    INSERT INTO ord
	    SELECT i, i || 'th';
	END LOOP;

	CREATE TEMPORARY TABLE IF NOT EXISTS dname( o int, t text );
	TRUNCATE dname;
	INSERT INTO dname
	VALUES
	(1, 'Monday'), (2, 'Tuesday'),(3,'Wednsday'),(4,'Thursday'),(5,'Friday'),(6,'Saturday'),(7,'Sunday');


	WHILE extract( month FROM current_date ) = m AND extract( year FROM current_date ) = y LOOP

	   IF extract( dow FROM current_date ) = 1 THEN
	      current_week := current_week + 1;
	   END IF;

	   RAISE INFO 'Date is %', current_date;

	   IF current_week = w AND extract( dow FROM current_date ) = d THEN
	      -- found
	      RAISE INFO 'Found on %', current_date;
	      SELECT t
	      INTO current_temp
	      FROM ord
	      WHERE o = w;

 	      current_result := 'The ' || current_temp;

	      SELECT t
	      INTO current_temp
	      FROM dname
	      WHERE o = extract( dow FROM current_date );

	      current_result := current_result || ' ' || current_temp || ' of year ' || y || ' is ' || extract(day from current_date);
	      RETURN current_result;
	   END IF;

	   SELECT current_date + 1
	   INTO current_date;


	END LOOP;

	RETURN 'Date not found';
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


I use two tables to keep the descriptive text of ordinality and days.


<a name="task2plpgsql"></a>
## PWC 237 - Task 2 - PL/PgSQL Implementation

A little cheating on this task: I call the PL/Perl function to handle arrays in a quick way.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc237.task2_plpgsql( nums int[] )
RETURNS int
AS $CODE$
	SELECT pwc237.task2_plperl( nums );
$CODE$
LANGUAGE sql;

```
<br/>
<br/>
