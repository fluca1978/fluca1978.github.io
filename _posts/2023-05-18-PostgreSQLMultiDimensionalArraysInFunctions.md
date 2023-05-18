---
layout: post
title:  "Multi-Dimensional Arrays in PostgreSQL"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A look at how PostgreSQL handles multi-dimensional arrays.

# Multi-Dimensional Arrays in PostgreSQL

PostgreSQL supports arrays of various types, and handles also multi-dimensional arrays.
*Except that it does not support multi-dimensional arrays*!

<br/>

Allow me to better explain.
Multi-dimensional arrays are *just* an array that contains other arrays. In this sense, PostgreSQL does not provide a pure native multi-dimensional array, even if you can specify them.

Let's see this in action by means of `pg_typeof`:

<br/>
<br/>
```sql
testdb=> select pg_typeof(  array[ array[ 1, 2 ],
                                   array[ 3, 4 ] ]::int[][] );
 pg_typeof
-----------
 integer[]
(1 row)

```
<br/>
<br/>

As you can see, the above *matrix* is repoted to be a single flat array.

<br/>

Consider now the following function, that accepts a multi-dimensional array and returns a table:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_matrix( int[][] )
RETURNS TABLE( a int, b int )
AS $CODE$
   my ( $matrix ) = @_;

   for my $row ( 0 .. $matrix->@* - 1 ) {
       for my $column ( 0 .. $matrix->[ $row ]->@* - 1 ) {
       	   return_next( { a => $row + 1,
	   		  b => $matrix->[ $row ]->[ $column ]
			} );
       }
   }

return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>

The above function, when invoked with a multi-dimensional array, works as expected:

<br/>
<br/>
```sql
testdb=> select *
         from f_matrix( array[ array[ 1, 2 ],
		                       array[ 3, 4 ] ]::int[][] );
 a | b
---+---
 1 | 1
 1 | 2
 2 | 1
 2 | 2
(4 rows)

```
<br/>
<br/>

However, if you inspect the function, *its signature clearly tells that the input parameter is a flat array*:

<br/>
<br/>
```sql
testdb=> \df f_matrix
                              List of functions
 Schema |   Name   |      Result data type       | Argument data types | Type
--------+----------+-----------------------------+---------------------+------
 public | f_matrix | TABLE(a integer, b integer) |   integer[]         | func
(1 row)

```
<br/>
<br/>


The same result, clearly, can be achieved by `plpgsql`, for example implementing the following function:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_matrix( m int[][] )
RETURNS TABLE( a int, b int )
AS $CODE$
DECLARE
   r int;
   c int;
BEGIN
	FOR r IN 1 .. array_length( m, 1 ) LOOP
	    FOR c IN 1 .. array_length( m, 2 ) LOOP
	    	a := r;
		    b := c;
		    RETURN NEXT;
	    END LOOP;
	END LOOP;

RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


In conclusion, PostgreSQL manages multi-dimensional arrays as flat lists, like what you would do in the C programming language.
This does not mean that you cannot use multi-dimensional arrays in an comfortable and efficient way, rather that you need to take into account how they are *handled* by the database engine, especially when passing them to a function.
