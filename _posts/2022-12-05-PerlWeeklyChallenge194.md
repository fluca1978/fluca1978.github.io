---
layout: post
title:  "Perl Weekly Challenge 194: regular expressions everywhere!"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 194: regular expressions everywhere!

This post presents my solutions to the [Perl Weekly Challenge 194](https://perlweeklychallenge.org/blog/perl-weekly-challenge-0194/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 194 - Task 1 - Raku](#task1)
- [PWC 194 - Task 2 - Raku](#task2)
- [PWC 194 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 194 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 194 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 194 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.


<a name="task1"></a>
## PWC 194 - Task 1 - Raku Implementation

The first task was about receiving an input from a digital clock, so in the form `hh:mm` where one digit was missing and substituted with a `?`. The task required to find out the max value such missing digit can assume.
The problem is quite simple, the only fact is that the missing digit can change its value depending on the position.
I decided to implement it by means of a few regular expressions able to catch where the digit is missing and which value it can assume:

<br/>
<br/>
```raku
sub MAIN( Str $what ) {

    given ( $what ) {
	when ( / ^ \? \d ':' \d ** 2 $ / )     { 2.say and exit  }
	when ( / ^ <[01]> \? ':' \d ** 2 $ / ) { 9.say and exit  }
	when ( / ^ 2 \? ':' \d ** 2 $ / )      { 3.say and exit  }
	when ( / ^ \d ** 2 ':' \? \d $ / )     { 5.say and exit  }
	when ( / ^\d ** 2 ':' \d \? $ / )      { 9.say and exit  }
    }
}

```
<br/>
<br/>

The PL/Perl implementation is a little smarter than that.


<a name="task2"></a>
## PWC 194 - Task 2 - Raku Implementation

The second task was about an input string of repeated characters. The task was about to find out if, removing a single character, all the remaining characters will have the same number of occurencies. It is not clear to me if the same frequency is required or not, so I assume only the same number of occurencies over the whole string.

<br/>
<br/>
```raku
sub MAIN( Str $what where { $what ~~ / ^ <[a..z]>+ $ / } ) {

    my $counter = Bag.new: $what.comb;

    "1".say and exit if ( $counter.values.max - $counter.values.min == 1
                         && $counter.keys.grep( { $counter{ $_ } == $counter.values.max } ) == 1 );

    "0".say;


}

```
<br/>
<br/>


The idea is quite simple: I count the occurencies of every character, keeping them into the `$counter` Bag. The program allows for only one character being different in size than the others, so there must be only one character that has a max value and such value must be greater than `1` than the other characters.


<a name="task1plperl"></a>
## PWC 194 - Task 1 - PL/Perl Implementation

A smarter approach than the Raku solution: I capture all the places where a digit can appear, and then elaborate what to do next:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc194.task1_plperl( text )
RETURNS int
AS $CODE$
 my ($what) = @_;

 if ( $what =~ / ^ ([\d?]) ([\d?]) : ([\d?]) ([\d?]) $ /x ) {
    if ( $1 eq '?' ) {
       return 9;
    }
    elsif ( $2 eq '?' ) {
      return 3 if $1 == 2;
      return 9;
    }
    elsif ( $3 eq '?' ) {
      return 5;
    }
    else {
      return 9;
    }
 }

return undef;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>



<a name="task2plperl"></a>
## PWC 194 - Task 2 - PL/Perl Implementation

Similar to the Raku implementation, with an *hand-made* Bag approach:

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc194.task2_plperl( text )
RETURNS int
AS $CODE$
  my ( $what ) = @_;

  my %counter;
  my ( $max, $min ) = ( 0, 0 );

  for ( split '', $what ) {
      $counter{ $_ }++;
      $min = $counter{ $_ } if ( ! $min || $min > $counter{ $_ } );
      $max = $counter{ $_ } if ( ! $max || $max < $counter{ $_ } );
  }

  return 0 if ( $max - $min != 1 );
  return 0 if ( grep( { $counter{ $_ } == $max }  keys %counter ) != 1 );
  return 1;

$CODE$
LANGUAGE plperl;

```
<br/>
<br/>




<a name="task1plpgsql"></a>
## PWC 194 - Task 1 - PL/PgSQL Implementation

An implementation following the PL/Perl approach: I match with a regular expression, then loop over the single letter/digit extracted. I need an index in order to know where I am within the string, as well as a previous character to keep in order to see what was the digit before the current one.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc194.task1_plpgsql( what text )
RETURNS int
AS $CODE$
DECLARE
	needle char;
	idx    int := 0;
	prev   char;
BEGIN
	FOREACH needle IN ARRAY regexp_match( what, '^([\d?])([\d?]):([\d?])([\d?])$' ) LOOP
	     IF needle <> '?' THEN
	        idx := idx + 1;
		prev := needle;
	        CONTINUE;
            END IF;

		IF idx = 0 THEN
		   RETURN 2;
		ELSEIF idx = 1 THEN
		    IF prev = '2' THEN
		       RETURN 3;
		    ELSE
		      RETURN 9;
		    END IF;
		ELSEIF idx = 2 THEN
		     RETURN 5;
		ELSE
		    RETURN 9;
		END IF;
	END LOOP;
RETURN NULL;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 194 - Task 2 - PL/PgSQL Implementation

I use a temporary table to count as a *bag*. I do insert every letter into the table, and in case of conflict, I issue an *upsert* to update the table and the letter counter. Then I'm able to process the `current_max` and `current_min`, as well as the number of letters having a `curent_max` counter.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc194.task2_plpgsql( what text)
RETURNS int
AS $CODE$
DECLARE
	t text;
	current_max int;
	current_min int;
	current_count int;
BEGIN
	CREATE TEMPORARY TABLE IF NOT EXISTS counter ( l char, c int, PRIMARY KEY(l) );
	TRUNCATE counter;

	FOR t IN SELECT v FROM regexp_split_to_table( what, '' ) v LOOP
	    INSERT INTO counter AS cnt ( l, c )
	    VALUES ( t, 1 )
	    ON CONFLICT (l)
	    DO UPDATE SET c = cnt.c + 1;
	END LOOP;

	SELECT max(c), min(c)
	INTO current_max, current_min
	FROM counter;

	IF current_max - current_min <> 1 THEN
	   RETURN 0;
	END IF;

	SELECT count(*)
	INTO current_count
	FROM counter
	WHERE c = current_max;

	IF current_count <> 1 THEN
	   RETURN 0;
	END IF;

	RETURN 1;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The conditions evaluated are the same as in the PL/Perl implementation.
