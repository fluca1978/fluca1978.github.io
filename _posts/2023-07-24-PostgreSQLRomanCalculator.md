---
layout: post
title:  "A PL/PgSQL Simple Roman Number Translator"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A way to decode a Roman number into an Arabic one and vice-versa using PL/PgSQL.

# A PL/PgSQL Simple Roman Number Translator

In the last [Weekly Challenge 227](https://theweeklychallenge.org/blog/perl-weekly-challenge-227/){:target="_blank"} the second task was about building a simple *Roman numbers calculator*. Since I usually try to implement those tasks also in PL/PgSQL (as well as in PL/Perl), I tried to implement such calculator and, along the path, I implemented a couple of simple functions to translate a number from and to roman notations.
<br/>
In this short post I explain how the two functions work.
<br/>
My approach is based on a lookup table that stores arabic and roman correspondencies for *special cases* and *base units*.

## The lookup table

I defined a lookup table, that can be in whatever schema you want, even temporary, and that is populated with base units and some special cases:

<br/>
<br/>
```sql
CREATE SCHEMA IF NOT EXISTS fluca1978;
CREATE  TABLE IF NOT EXISTS fluca1978.roman( r text, n int, repeatable boolean );

TRUNCATE TABLE fluca1978.roman;

INSERT INTO fluca1978.roman
VALUES
('I', 1, true )
,( 'IV', 4, false )
,( 'V', 5, false )
,( 'IX', 9, false )
,( 'X', 10, true )
,( 'XL', 40, false )
,( 'L', 50, false )
,( 'XC', 90, false )
,( 'C', 100, true )
,( 'CD', 400, false )
,( 'D', 500, false )
,( 'CM', 900, false )
,( 'M', 1000, true );


```
<br/>
<br/>


The `r` field holds the roman value for a number `n`, while the `repeatable` flag indicates if the number can be repeated consequently in the same stirng. For example, `I` can be repeated to form `III`, while `IV` cannot be repeated into `IVIV`. This will be useful during validation.


## Validating a Roman String

The following function perform the minimal validation for a given input string that is supposed to be a roman number:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
fluca1978.validate_roman( r text )
RETURNS boolean
STRICT
AS $CODE$
DECLARE
	current_record fluca1978.roman%rowtype;
	rx text;
	matches int;
BEGIN


	FOR current_record IN SELECT * FROM fluca1978.roman ORDER BY n DESC LOOP
	    RAISE DEBUG 'Iterating over Roman value % = %', current_record.r, current_record.n;

	    matches := 0;
	    rx := format( '^%s', current_record.r );

	    WHILE r ~ rx LOOP
	    	  matches := matches + 1;
		  RAISE DEBUG 'Input string % -> % matches the Roman value %', r, matches, current_record.r;

		  IF NOT current_record.repeatable AND matches > 1 THEN
		     RAISE DEBUG 'Roman symbol % cannot be repeated!', current_record.r;
		     RETURN false;
		  END IF;

		  r := regexp_replace( r, rx, '' );
		  EXIT WHEN length( r ) = 0;
	    END LOOP;

 	   EXIT WHEN length( r ) = 0;
	END LOOP;

	IF length( r ) > 0 THEN
	   RETURN false;
	END IF;

	RETURN true;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The idea is simple: I order the lookup table in descending order, so from the biggest value to the smallest one.
At each iteration, I search fi the current roman string starts with the roman letter (or couple of letters). If that is the case, I keep track of how many `matches` I've found, then remove the roman symbol from the beginning of the string.
Then I see if the same letter/symbol can be found in the beginning of the string, and if so, I ensure it is a repeatable value, otherwise there is an error.
If everything goes well, the ending string `r` will be empty due to the substitutions, otherwise if some characters remain then the string is wrong.
That happens, for example, when the roman values on the right are biggest than those on the left.



## Converting from Roman to Arabic

The following function does the convertion starting from a Roman string:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
fluca1978.from_roman( r text )
RETURNS int
STRICT
AS $CODE$
DECLARE
	v int := 0;
	current_record fluca1978.roman%rowtype;
	rx text;
BEGIN
	IF r = '' THEN
	   RETURN 0;
	END IF;

	IF NOT fluca1978.validate_roman( r ) THEN
	   RETURN 0;
	END IF;

	FOR current_record IN SELECT * FROM fluca1978.roman ORDER BY n DESC LOOP
	     RAISE DEBUG 'Iterating over Roman value % = %', current_record.r, current_record.n;

	     rx := format( '^%s', current_record.r );
	    WHILE r ~ rx LOOP
		RAISE DEBUG 'Input string % matches the Roman value %', r, current_record.r;

	        v := v + current_record.n;
	        r := regexp_replace( r, rx, '' );
	    END LOOP;
	END LOOP;

	RAISE DEBUG 'Converted value is %', v;
	RETURN v;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

It is really similar to the validating function: it iterates on each part of the string searching to decode as the biggest possible value in the roman lookup table.
<br/>
<br/>
It is possible to see the workflow of the function by means of using the `DEBUG` log level:

<br/>
<br/>
```sql
testdb=> set client_min_messages to debug;
SET
testdb=> select fluca1978.from_roman( 'MCMLXXVIII' );
DEBUG:  Iterating over Roman value M = 1000
DEBUG:  Input string MCMLXXVIII matches the Roman value M
DEBUG:  Iterating over Roman value CM = 900
DEBUG:  Input string CMLXXVIII matches the Roman value CM
DEBUG:  Iterating over Roman value D = 500
DEBUG:  Iterating over Roman value CD = 400
DEBUG:  Iterating over Roman value C = 100
DEBUG:  Iterating over Roman value XC = 90
DEBUG:  Iterating over Roman value L = 50
DEBUG:  Input string LXXVIII matches the Roman value L
DEBUG:  Iterating over Roman value XL = 40
DEBUG:  Iterating over Roman value X = 10
DEBUG:  Input string XXVIII matches the Roman value X
DEBUG:  Input string XVIII matches the Roman value X
DEBUG:  Iterating over Roman value IX = 9
DEBUG:  Iterating over Roman value V = 5
DEBUG:  Input string VIII matches the Roman value V
DEBUG:  Iterating over Roman value IV = 4
DEBUG:  Iterating over Roman value I = 1
DEBUG:  Input string III matches the Roman value I
DEBUG:  Input string II matches the Roman value I
DEBUG:  Input string I matches the Roman value I
DEBUG:  Converted value is 1978
 from_roman
------------
       1978
(1 row)


```
<br/>
<br/>

As you can see, at every iteration the function removes the leftmost letter from the string and continues to see what it can find next.
<br/>
The matching is performed by building a regular expression as condition to the `WHILE` loop: the condition has the *begin at string* anchor `^` followed by whatever roman symbole is in the current record out of the lookup table. The special `EXIT` part ensures that there cannot be repetitions of two letetrs symbols. For example you cannot express `IVIV` as `8`, so once `IV` is encountered, the `WHILE` knows it can safely exit the loop.


## Converting From Arabic to Roman

The following function does the opposite: converts an integer into a roman string.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
fluca1978.to_roman( n int )
RETURNS text
STRICT
AS $CODE$

DECLARE
	roman_value text := '';
    current_record fluca1978.roman%rowtype;
BEGIN
	IF n <= 0 THEN
		RAISE DEBUG 'Cannot convert zero!';
		RETURN NULL;
	END IF;

	FOR current_record IN SELECT * FROM fluca1978.roman ORDER BY n DESC LOOP

	    WHILE n >= current_record.n LOOP
			RAISE DEBUG 'The value % is greater than % so appending a %', n, current_record.n, current_record.r;
			roman_value := roman_value || current_record.r;
			n := n - current_record.n;
			EXIT WHEN length( current_record.r ) = 2;
	    END LOOP;
	END LOOP;

	RAISE DEBUG 'Computed value is %', roman_value;
	RETURN roman_value;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

Again, thanks to the debug output it is easy to understand the workflow of the converter:

<br/>
<br/>
```sql
testdb=> select fluca1978.to_roman( 1978 );
DEBUG:  The value 1978 is greater than 1000 so appending a M
DEBUG:  The value 978 is greater than 900 so appending a CM
DEBUG:  The value 78 is greater than 50 so appending a L
DEBUG:  The value 28 is greater than 10 so appending a X
DEBUG:  The value 18 is greater than 10 so appending a X
DEBUG:  The value 8 is greater than 5 so appending a V
DEBUG:  The value 3 is greater than 1 so appending a I
DEBUG:  The value 2 is greater than 1 so appending a I
DEBUG:  The value 1 is greater than 1 so appending a I
DEBUG:  Computed value is MCMLXXVIII
  to_roman
------------
 MCMLXXVIII

```
<br/>
<br/>



## Caching Results

It is, clearly, very simple to define a materialized view or a cache table to handle all values for a faster lookup.
As an example, imagine to create a table that serves as a cache:

<br/>
<br/>
```sql
CREATE TABLE IF NOT EXISTS fluca1978.roman_cache_table( n int, r text );
TRUNCATE TABLE fluca1978.roman_cache_table;

INSERT INTO fluca1978.roman_cache_table( n, r )
SELECT n, r
FROM   fluca1978.roman
ORDER BY n;

```
<br/>
<br/>

and then a function that, given a number, tries to understand if the caching table contains such a number, otherwise populates the table from the last found index to the given one

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
fluca1978.roman_cache( x int )
RETURNS text
STRICT
AS $CODE$
DECLARE
	max_cached_value int;
	i int;
	v text;
BEGIN
	SELECT max( n )
	INTO max_cached_value
	FROM fluca1978.roman_cache_table;

	RAISE DEBUG 'Max cached value % and looking for %', max_cached_value, x;

	IF max_cached_value IS NULL OR x > max_cached_value THEN
	   IF max_cached_value IS NULL THEN
	      max_cached_value := 1;
	   END IF;
	   RAISE DEBUG 'Repopulating the cache from % to %', max_cached_value, x;

	   FOR i IN max_cached_value + 1 .. x LOOP
	   	   INSERT INTO fluca1978.roman_cache_table( n, r )
	   	   SELECT i, fluca1978.to_roman( i );
	   END LOOP;
	END IF;

	SELECT r
	INTO v
	FROM fluca1978.roman_cache_table
	WHERE n = x;

	RETURN v;
END
$CODE$
LANGUAGE plpgsql;




```
<br/>
<br/>


When you query the above function, the system inspects the `roman_cache_table` for the asked arabic number, and the number is in there it returns it. If the number is greater than the max value within the caching table, the function populates the table up to the given number.





# Conclusions

With some patient and a few iterations, it is possible to create a fully functional Roman Number Converter, and hence also a calculator.
Clearly, this kind of task is much more simpler with Perl (and PL/Perl), but PL/PgSQL can handle it too with a littlemore verbosity.
Code from the above examples can be found on my [Github repository](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/roman-converter/roman-converter.sql ){:target="_blank"}.
