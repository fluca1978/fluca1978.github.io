---
layout: post
title:  "Functions to Validate User's Input"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PostgreSQL 16 introduces a couple of functions to validate user's input.

# Functions to Validate User's Input

PostgreSQL 16 introduces a couple of new embedded functions: `[pg_input_is_valid](https://www.postgresql.org/docs/16/functions-info.html#FUNCTIONS-INFO-VALIDITY-TABLE){:target="_blank"}`  and `[pg_input_error_info](https://www.postgresql.org/docs/16/functions-info.html#FUNCTIONS-INFO-VALIDITY-TABLE){:target="_blank"}`.

Both the functions accepts a couple of strings, the first one being the value to be validated, and the second one being the type to which *you want to cast the value*. This can be useful because you can check ahead of time if a given data type (expressed as a string) can be converted into a specific data type without raising an exception.

The first use case that comes into my mind is the conversion of some stringified date into an effective date, for example when importing data from an external source like a text file. Let's see this in action:

<br/>
<br/>
```sql
testdb=> select * from pg_input_is_valid( '1978-07-19', 'timestamp' );
 pg_input_is_valid
-------------------
 t
(1 row)

testdb=> select * from pg_input_error_info( '1978-07-19', 'timestamp' );
 message | detail | hint | sql_error_code
---------+--------+------+----------------
         |        |      |
(1 row)


```
<br/>
<br/>

With a valid date, the `pg_input_is_valid` function returns true and the `pg_input_error_info` does not return any row.
But what happens if the date is in a wrong format?

<br/>
<br/>
```sql
testdb=> \x
Expanded display is on.
testdb=> select * from pg_input_is_valid( '1978-19-07', 'timestamp' );
-[ RECORD 1 ]-----+--
pg_input_is_valid | f

testdb=> select * from pg_input_error_info( '1978-19-07', 'timestamp' );
-[ RECORD 1 ]--+--------------------------------------------------
message        | date/time field value out of range: "1978-19-07"
detail         |
hint           | Perhaps you need a different "datestyle" setting.
sql_error_code | 22008

```
<br/>
<br/>

As you can see from the above example, passing a wrong date/time format raises the error, and thanks to these functions we are now able to discover ahead of its usage what the problem could be.

Another example, just to clarify more:

<br/>
<br/>
```sql
testdb=> select pg_input_error_info( '4 months', 'interval' );
-[ RECORD 1 ]-------+------
pg_input_error_info | (,,,)

testdb=> select pg_input_error_info( '4 mesi', 'interval' );
-[ RECORD 1 ]-------+---------------------------------------------------------------
pg_input_error_info | ("invalid input syntax for type interval: ""4 mesi""",,,22007)

```
<br/>
<br/>


It is therefore quite easy to use such checks into your own function:

<br/>
<br/>
```sql
testdb=> CREATE OR REPLACE FUNCTION input_check( t text[] )RETURNS int
AS $CODE$
DECLARE
  current text; ok int := 0;  e text;
BEGIN
  FOREACH current IN ARRAY t LOOP
    IF pg_input_is_valid( current, 'date' ) THEN
       ok := ok + 1;
    ELSE
       SELECT message
       INTO e
       FROM pg_input_error_info( current, 'date' );
       RAISE DEBUG 'Skipping [%] because is not valid: %', current, e;
   END IF;
  END LOOP;

  RETURN ok;
END
$CODE$
LANGUAGE plpgsql;
CREATE FUNCTION


```
<br/>
<br/>


that, once invoked with the following input, provides the result as shown below:

<br/>
<br/>
```sql
testdb=> select input_check( array[ '2023-09-25', 'luca', '0001-01-01', 'Sat 23 Sep 2023', 'Feb 30 2023' ] );
DEBUG:  Skipping [luca] because is not valid: invalid input syntax for type date: "luca"
DEBUG:  Skipping [Feb 30 2023] because is not valid: date/time field value out of range: "Feb 30 2023"
 input_check
-------------
           3
(1 row)

```
<br/>
<br/>
