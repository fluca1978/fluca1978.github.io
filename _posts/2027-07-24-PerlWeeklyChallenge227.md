---
layout: post
title:  "Perl Weekly Challenge 227: Roman Fridays"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
- perl
permalink: /:year/:month/:day/:title.html
---
My personal solutions to the Perl Weekly Challenge.

# Perl Weekly Challenge 227: Roman Fridays

This post presents my solutions to the [Perl Weekly Challenge 227](https://perlweeklychallenge.org/blog/perl-weekly-challenge-227/){:target="_blank"}.
<br/>
I keep doing the Perl Weekly Challenge in order to mantain my coding skills in good shape, as well as in order to learn new things, with particular regard to Raku, a language that I love.
<br/>
This week, I solved the following tasks:

<br/>
- [PWC 227 - Task 1 - Raku](#task1)
- [PWC 227 - Task 2 - Raku](#task2)
- [PWC 227 - Task 1 in PostgreSQL PL/Perl](#task1plperl)
- [PWC 227 - Task 2 in PostgreSQL PL/Perl](#task2plperl)
- [PWC 227 - Task 1 in PostgreSQL PL/PgSQL](#task1plpgsql)
- [PWC 227 - Task 2 in PostgreSQL PL/PgSQL](#task2plpgsql)

The PL/Perl implementations are very similar to a pure Perl implementation, even if the PostgreSQL environment could involve some more constraints. Similarly, the PL/PgSQL implementations help me keeping my PostgreSQL programming skills in good shape.

# Raku Implementations

<a name="task1"></a>
## PWC 227 - Task 1 - Raku Implementation

The first task was very simple: given an year, find out how many *Friday 13th* there are in such an year.

<br/>
<br/>
```raku
sub MAIN( Int $year where { 1753 <= $year <= 9999 }, Bool :$verbose = True ) {
    my @fridays;
    for 1 .. 12 -> $month {
		my $day = Date.new( year => $year, day => 13, month => $month );
		@fridays.push: $day if ( $day.day-of-week == 5 );
    }

    @fridays.elems.say;
    @fridays.join( ', ' ).say if $verbose;
}

```
<br/>
<br/>

The idea is simple: I iterate on the twelve months of an year and build an `Date` object pointing at the day `13` of such a month. If that day is the fifth in the week, i.e., a Friday, I add it to the `@fridays` array. In the ends, I can count the number of elements in the `@fridays` array and even print them out in the case the program has been invoked with a verbose mode.


<a name="task2"></a>
## PWC 227 - Task 2 - Raku Implementation

Implement a simple Roman based calculator.

<br/>
<br/>
```raku
sub rom-to-num($r) {
    [+] gather $r.uc ~~ /
        ^
        [
        | M  { take 1000 }
        | CM { take 900 }
        | D  { take 500 }
        | CD { take 400 }
        | C  { take 100 }
        | XC { take 90 }
        | L  { take 50 }
        | XL { take 40 }
        | X  { take 10 }
        | IX { take 9 }
        | V  { take 5 }
        | IV { take 4 }
        | I  { take 1 }
        ]+
        $
    /;
}


my %symbols =
    1 => 'I',
    4 => 'IV',
    5 => 'V',
    9 => 'IX',
    10 => 'X',
    40 => 'XL',
    50 => 'L',
    90 => 'XC',
    100 => 'C',
    400 => 'CD',
    500 => 'D',
    900 => 'CM',
    1000 => 'M'
;

sub arabic-to-roman( $number ) {
    return ''  if $number == 0;

    for %symbols.keys.sort( { $^b <=> $^a } ) {
	if $number >= $_ {
	    return %symbols{ $_ } ~ arabic-to-roman( $number - $_ );
	}
    }

}

sub MAIN( *@s where { @s.elems == 3 } ) {

    my @operands = rom-to-num( @s[ 0 ] ), rom-to-num( @s[ 2 ] );
    my $result;
    given ( @s[ 1 ] ) {
		when '+' { $result = [+] @operands; }
		when '-' { $result = [-] @operands; }
		when '*' { $result = [*] @operands; }
		when '/' { $result = [/] @operands; }

    }

    exit if $result <= 0;
    say arabic-to-roman( $result );
}

```
<br/>
<br/>


The function `rom-to-num` is a well web-searchable implementation of a translator from roman to arabic numbers.
The `arabic-to-roman` does the opposite and computes an integer value for a given roman number string.
The `MAIN` implements the calculator and exploits a `given`/`when` implementation where the `$result` of the computation is computed by means of a recuction operator. The `operands` array contains the arabic converted input numbers.
In the end, `arabic-to-roman` is used to convert the `$result` back in a roman format.


# PL/Perl Implementations


<a name="task1plperl"></a>
## PWC 227 - Task 1 - PL/Perl Implementation

Similar to the Raku implementation, but increments the month by one unit at every iteration, and stops whenever the `month` is greater than `12` (impossible) or the `year` has changed.

<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc227.task1_plperl( int )
RETURNS int
AS $CODE$
   my ( $year ) = @_;
   my $fridays = 0;

   use DateTime;
   my $day = DateTime->new( year => $year, day => 13, month => 1 );
   while ( $day->month <= 12 && $day->year == $year ) {
   	 $fridays++ if ( $day->day_abbr eq 'Fri' );
	 $day->add(  months => 1  );
   }

   return $fridays;
$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

Requires to use an untrusted language because the need of `DateTime` library.

<a name="task2plperl"></a>
## PWC 227 - Task 2 - PL/Perl Implementation

Similar to the Raku approach, exploits two anonymous functions to perform the conversion from/to roman. Exploits the `Sub::Recursive` module to implement recursion within an anonymous function.


<br/>
<br/>
```perl
CREATE OR REPLACE FUNCTION
pwc227.task2_plperl( text, text, text )
RETURNS text
AS $CODE$
   use v5.20;
   my ( $left, $operator, $right ) = @_;

   my $symbols = {
          1 => 'I',
	  4 => 'IV',
	  5 => 'V',
	  9 => 'IX',
	 10 => 'X',
	 40 => 'XL',
	 50 => 'L',
	 90 => 'XC',
	 100 => 'C',
	 400 => 'CD',
	 500 => 'D',
	 900 => 'CM',
	1000 => 'M'
      };


   my $unsymbols = {};
   $unsymbols->{ $symbols->{ $_ } } = $_   for ( keys $symbols->%* );



   use Sub::Recursive;

   my $to_roman = recursive {
      my ( $number ) = @_;




      return '' if ! $number;

      for my $arabic ( sort  { $b <=> $a } keys $symbols->%* ) {
      	  if ( $number >= $arabic ) {
	     return $symbols->{ $arabic } . $REC->( $number - $arabic );
	  }
      }

   };


   my $to_arabic = sub {
      my ( $number ) = @_;
      my $value = 0;
      for my $roman ( reverse sort keys $unsymbols->%* ) {
      	  $value += $unsymbols->{ $roman } while $number =~ s/^$roman//i;
      }

      return $value;
   };


   my $result = 0;
   given ( $operator ) {
   	 when (/\+/) { $result = $to_arabic->( $left ) + $to_arabic->( $right ); }
     when (/\-/) { $result = $to_arabic->( $left ) - $to_arabic->( $right ); }
   	 when (/\//) { $result = $to_arabic->( $left ) / $to_arabic->( $right ); }
   	 when (/\*/) { $result = $to_arabic->( $left ) * $to_arabic->( $right ); }
   }

   return undef if ( $result < 1 );
   return $to_roman->( $result );

$CODE$
LANGUAGE plperlu;

```
<br/>
<br/>

Again, it requires an untrusted language to load modules.


# PostgreSQL Implementations

<a name="task1plpgsql"></a>
## PWC 227 - Task 1 - PL/PgSQL Implementation

Use `extract` to get the day of the week on a month based iteration.


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
pwc227.task1_plpgsql( y int)
RETURNS int
AS $CODE$
DECLARE
	fridays int := 0;
	m int;
BEGIN
	FOR m IN 1 .. 12 LOOP
	    IF extract( dow FROM make_date( y, m, 13 ) ) = 5 THEN
	       fridays := fridays + 1;
	    END IF;
	END LOOP;

	RETURN fridays;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>



<a name="task2plpgsql"></a>
## PWC 227 - Task 2 - PL/PgSQL Implementation


Uses different utility functions and a lookup table to perform the conversion from/to roman numbers.


<br/>
<br/>
```sql
CREATE  TABLE IF NOT EXISTS pwc227.roman( r text, n int );

TRUNCATE TABLE pwc227.roman;

INSERT INTO pwc227.roman
VALUES
('I', 1 )
,( 'IV', 4 )
,( 'V', 5 )
,( 'IX', 9 )
,( 'X', 10 )
,( 'XL', 40 )
,( 'L', 50 )
,( 'XC', 90 )
,( 'C', 100 )
,( 'CD', 400 )
,( 'D', 500 )
,( 'CM', 900 )
,( 'M', 1000 );




CREATE OR REPLACE FUNCTION
pwc227.to_roman( n int )
RETURNS text
AS $CODE$

DECLARE
	roman_value text := '';
        current_record pwc227.roman%rowtype;
BEGIN
	IF n <= 0 THEN
	   RETURN NULL;
	END IF;

	IF n = 1 THEN
	   RETURN 'I';
	END IF;

	FOR current_record IN SELECT * FROM pwc227.roman ORDER BY n DESC LOOP

	    WHILE n >= current_record.n LOOP
	       roman_value := roman_value || current_record.r;
	       n := n - current_record.n;
	    END LOOP;
	END LOOP;

	RETURN roman_value;
END
$CODE$
LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION
pwc227.from_roman( r text )
RETURNS int
AS $CODE$
DECLARE
	v int := 0;
	current_record pwc227.roman%rowtype;
BEGIN
	FOR current_record IN SELECT * FROM pwc227.roman ORDER BY n DESC LOOP
	    WHILE r ~ ( '^' || current_record.r)	   LOOP
	       v := v + current_record.n;
	       r := regexp_replace( r, '^' || current_record.r, '' );
	    END LOOP;
	END LOOP;

	RETURN v;
END
$CODE$
LANGUAGE plpgsql;



CREATE OR REPLACE FUNCTION
pwc227.task2_plpgsql( a text, op text, b text )
RETURNS text
AS $CODE$
DECLARE
	v int;
BEGIN
	IF op = '+' THEN
	   v := pwc227.from_roman( a ) + pwc227.from_roman( b );
	ELSIF op = '-' THEN
	   v := pwc227.from_roman( a ) - pwc227.from_roman( b );
	ELSIF op = '*' THEN
	  v := pwc227.from_roman( a ) * pwc227.from_roman( b );
	ELSIF op = '/' THEN
   	  v := pwc227.from_roman( a ) / pwc227.from_roman( b );
	END IF;

	RETURN pwc227.to_roman( v );

END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The implementation is the same as PL/Perl, but using a lookup table allows for a simpler lookup in both the utility functions.
