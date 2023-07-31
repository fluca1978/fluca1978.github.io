---
layout: post
title:  "A Possible Way to Implement a Shift Function in PL/PgSql"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Creating a shift-like function for manipulating arrays in PL/PgSQL.

# A Possible Way to Implement a Shift Function in PL/PgSql

PostgreSQL does support arrays in a very excellent way, but it does not provide a `shift` like function.
A `shift` function takes an array as input and removes the first (left-most) element from the array.
This is quite simple to do in PostgreSQL, since *array slices* are easy to implement. However, a slice returns the modified (shifted) array, not the shifted element.

<br/>
<br/>

It is possible to implement a very simple function in PL/PgSQL that accepts an array of anytype and returns a table like multi-cardinality result set, with the element removed and the resulting array. The following is a straightforward implementation:


<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
shift( a anyarray,
       loops int default 1,
       emit_intermediate boolean default false )
RETURNS TABLE( head text, tail anyarray, step int )
AS $CODE$
BEGIN
	-- check that the array is good and has
	-- at least one element
	IF a IS NULL OR array_length( a, 1 ) < 1 THEN
	   RETURN;
	END IF;

	-- if the array has less elements that those
	-- to shift, do only the max available shifting
	IF loops > array_length( a, 1 ) THEN
	   loops := array_length( a, 1 );
	END IF;

	-- initialize the returning array and the
	-- number of steps
	tail := a;
	step := 1;

	WHILE loops > 0 LOOP
		head := tail[ 1 ];
		tail := tail[ 2 : array_length( tail, 1 ) ];

		IF emit_intermediate OR loops = 1 THEN
		   RETURN NEXT;
		END IF;

		loops := loops - 1;
		step  := step + 1;
	END LOOP;

	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>


The idea is quite simple: the function accepts the array to shift and, optionally, the number of times the `shift` operation has to be performed, as well as a flag to indicate if the intermediate steps have to be emitted. The function iterates over the number of shifts to be performed, and removes the head element from the array.
<br/>
As an example:

<br/>
<br/>
```sql
testdb=> select * from shift( array[ 'cat', 'dog', 'parrot' ] );
 head |     tail     | step
------+--------------+------
 cat  | {dog,parrot} |    1
(1 row)

```
<br/>
<br/>

As you can see, the first element of the array, `cat`, is removed from the array that remains as `{dog,parrot]`; the removed (shifted) element is returned as `head` and the remaining array as `tail`. The `step` column indicates at which iteration the result refers to.

<br/>
As another example, let's do a shift twice, emitting intermediate results in the meantime:

<br/>
<br/>
```sql
testdb=> select * from shift( array[ 'cat', 'dog', 'parrot' ], 2, true );
 head |     tail     | step
------+--------------+------
 cat  | {dog,parrot} |    1
 dog  | {parrot}     |    2
(2 rows)

```
<br/>
<br/>

As you can see, this time two tuples are emitted. The first one (`step = 1`), the `cat` element is removed and the `{dog,parrot}` array is returned. At the second iteration (`step = 2`), the `dog` element is removed from the array and the remaining `{parrot}` is returned. Without emitting the intermediate results, the function returns always a single tuple:

<br/>
<br/>
```sql
testdb=> select * from shift( array[ 'cat', 'dog', 'parrot' ], 2 );
 head |   tail   | step
------+----------+------
 dog  | {parrot} |    2
(1 row)

```
<br/>
<br/>

## Usage Example

As an example, it is possible to use the `shift` function into another PL/PgSQL piece of code using an assignment, for example a `SELECT INTO`:

<br/>
<br/>
```sql
DO LANGUAGE plpgsql $CODE$
DECLARE
	a text[];
	h text;
	I INT;
BEGIN
	a := array[ 'alfa', 'beta', 'gamma', 'delta' ];

	FOR i IN 1 .. 2 LOOP
		SELECT head, tail
		INTO h, a
		FROM shift( a );

		RAISE INFO 'Removed <%> = %', h, a;
	END LOOP;
END
$CODE$

```
<br/>
<br/>

that produces the following dummy output:

<br/>
<br/>
```sql
NFO:  Removed <alfa> = {beta,gamma,delta}
INFO:  Removed <beta> = {gamma,delta}
DO

```
<br/>
<br/>


## Efficiency Considerations

The function `shift`, as it is implemented, is not really efficient because it performs a set of iterations over the given array. In the case there is no need to emit the intermediate stages, it is possible to shrink the function as a couple of operations, mainly an array slice.
The following is a possible, slightly more efficient implementation:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
shiftx( a anyarray,
        loops int default 1 )
RETURNS TABLE( head text, tail anyarray )
AS $CODE$
BEGIN
	-- check that the array is good and has
	-- at least one element
	IF a IS NULL OR array_length( a, 1 ) < 1 THEN
	   RETURN;
	END IF;

	-- if the array has less elements that those
	-- to shift, do only the max available shifting
	IF loops > array_length( a, 1 ) THEN
	   loops := array_length( a, 1 );
	END IF;

	-- initialize the returning array
	-- and the head of the last element
	head := a[ loops ];
	tail := a[ 1 + loops : array_length( a, 1 ) ];

	RETURN NEXT;
	RETURN;
END
$CODE$
LANGUAGE plpgsql;

```
<br/>
<br/>

The above implementation returns a single tuple, where the `head` is the last removed element at the `loops` offset, while the `tail` is the array slice removed from the array itself. Since this implementation does not perform any iteration, it can be a little faster on multi-occurencies shifts.

<br/>
It is possible to perform a quick and dirty test about performances with a `DO` code that performs a few thousands of iterations comparing the results:

<br/>
<br/>
```sql
DO LANGUAGE plpgsql
$CODE$
DECLARE
	a text[];
	ts_begin timestamp;
	ts_end   timestamp;
	iter     int;
	i        int;
BEGIN

	iter := 2000;

	-- initialize the array
	SELECT '{' || string_agg( v::text, ',' ) || '}'
	INTO a
	FROM generate_series( 1, iter ) v;

	ts_begin := clock_timestamp();
	FOR i IN 1 .. iter LOOP
	    PERFORM shift( a, iter / 2 );
	END LOOP;
	ts_end := clock_timestamp();

	RAISE INFO 'Using shift for % iteration over % elements = %',
	      	   iter,
		   array_length( a, 1 ),
		   ( ts_end - ts_begin );


	ts_begin := clock_timestamp();
	FOR i IN 1 .. iter LOOP
	    PERFORM shiftx( a, iter / 2 );
	END LOOP;
	ts_end := clock_timestamp();

	RAISE INFO 'Using shiftx for % iteration over % elements = %',
	      	   iter,
		   array_length( a, 1 ),
		   ( ts_end - ts_begin );


END
$CODE$;
```
<br/>
<br/>

The above builds an array of `2000` elements and performs `2000` shifts with a cardinality of `100`, that is removes the first 1000 elements from the array and loops 2000 times.
<br/>
The results are the following, clearly depending on the machine they are run:

<br/>
<br/>
```sql
INFO:  Using shift for 2000 iteration over 2000 elements = 00:01:04.686821
INFO:  Using shiftx for 2000 iteration over 2000 elements = 00:00:00.055818
DO
Time: 64746,850 ms (01:04,747)

```
<br/>
<br/>

As you can see, the `shiftx` is clearly faster than the `shift` iterating version. This is clearly evenmore understandable if we raise the number of iterations and shifts to `5000`:

<br/>
<br/>
```sql
INFO:  Using shift for 5000 iteration over 2505 elements = 00:05:40.513576
INFO:  Using shiftx for 5000 iteration over 2505 elements = 00:00:00.078997
DO
Time: 340597,743 ms (05:40,598)

```
<br/>
<br/>



# Conclusions

Thanks to array slices it is simple enough to implement a `shift` like functionality in PL/PgSQL. The problem of the approach described here is that there is no easy way to modify the array in place, for example thru a reference, so the functions need to return a compound result like a table.
