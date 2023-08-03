---
layout: post
title:  "A Possible Way to Implement a Shift Function in PL/PgSql (part 2)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Creating a shift-like function for manipulating arrays in PL/PgSQL.

# A Possible Way to Implement a Shift Function in PL/PgSql (part 2)

After my post about [how to implement a `shift` like operation in PostgreSQL](https://fluca1978.github.io/2023/08/01/PostgreSQLShiftArray.html){:target="_blank"}
I got some comments and suggestions, most notably a *pure SQL implementation* provided by **Stefan Stefanov**, tho whom belongs the credits for the solution, and that allowed me to explain in this (second) article on the subject.
<br/>
In the following you will find the **Stefan Stefanov**'s solution, a PL/Perl implementation I made in the meantime, and a little benchmarking to see how all the approaches compare to each other.


## A pure SQL Implementation (credits to Stefan Stefanov)

The following is the function proposed by **Stefan Stefanov**:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION array_shift(arr anyarray, loops integer DEFAULT 1)
 RETURNS TABLE(head anyelement, tail anyarray)
 LANGUAGE sql
AS $function$

   with arr_tbl(el, arr_index) as (
       select * from unnest(arr) with ordinality
   )
   select (select el from arr_tbl where arr_index = loops), 
          (select array_agg(el order by arr_index) 
		          from arr_tbl where arr_index > loops);
$function$				  
```	
<br/>
<br/>


As you can see, this is a very clever approach that exploits only `SELECT` statements to get the final result.
The `arr_tbl` CTE *explodes* the array by means of the PostgreSQL builting `unnest` function, and returns the array as a table with the `ordinality`, that is an automatically added column that works as a row number.
The output of the CTE is similar to the following one:

<br/>
<br/>
```sql
testdb=> select * from unnest( array['alfa','beta', 'gamma' ] ) with ordinality;
 unnest | ordinality 
--------+------------
 alfa   |          1
 beta   |          2
 gamma  |          3
(3 rows)
	
```
<br/>
<br/>

The main `SELECT` performs the selection of two different columns, both extracted by a subquery.
The first subquery extracts the *last* element from the `shift` operation, that is the one with the ordinality (i.e., row number) equal to the number of loops. Assuming `loops = 2`, it extracts the `beta` value from the above table. This is what I called the `head` in my functions.
<br/>
The other subquery extracts the elements with the ordinality greater than the number of shifts, that is all the remaining elements, and then re-agrgegates them into an array by means of the PostgreSQL builtin `array_agg` function.

<br/>
The beauty of this idea is that everything is built on top of queries, that is the array is transformed into a table and then back into an array, but all the computation is done as *cascading `SELECT`*.





## A PL/Perl implementation

Since Perl comes with a *natural* `shift` operator, why not using it as a wrapper to shift a PostgreSQL array?
<br/>
The only drawback of this approach is that PL/Perl does not allow to pass an `anyarray` argument to a function, so there is the need to make an array-specific implementation:

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
shift_plperl(  text[],
                int default 1 )
RETURNS TABLE( head text, tail text[] )
AS $CODE$
   my ( $array, $loops ) = @_;
   my ( $head );

   $head = shift $array->@* for ( 1 .. $loops );
   return_next( { head => $head, tail => $array } );
   return undef;
$CODE$
LANGUAGE plperl;

```
<br/>
<br/>





## The tests

The tests have been done, as in the previous post, with a block code similar to the following:

<br/>
<br/>
```sql
testdb=> DO LANGUAGE plpgsql
$CODE$
DECLARE
        a text[];
        ts_begin timestamp;
        ts_end   timestamp;
        iter     int;
        i        int;
BEGIN

        iter := 7000;

        -- initialize the array
        ts_begin := clock_timestamp();
        SELECT '{' || string_agg( v::text, ',' ) || '}'
        INTO a
        FROM generate_series( 1, iter / 2 + 5 ) v;
        ts_end := clock_timestamp();

        RAISE INFO 'Array allocation = %', ( ts_end - ts_begin );

		RAISE INFO 'Using shift for % iterations over % elements = %',
                   iter,
                   array_length( a, 1 ),
                   ( ts_end - ts_begin );


        ts_begin := clock_timestamp();
        FOR i IN 1 .. iter LOOP
            PERFORM shiftx( a, iter / 2 );
        END LOOP;
        ts_end := clock_timestamp();

        ts_begin := clock_timestamp();
        FOR i IN 1 .. iter LOOP
            PERFORM shiftx( a, iter / 2 );
        END LOOP;
        ts_end := clock_timestamp();

        RAISE INFO 'Using shiftx for % iterations over % elements = %',
                   iter,
                   array_length( a, 1 ),
                   ( ts_end - ts_begin );



        ts_begin := clock_timestamp();
        for i in 1 .. iter loop
                perform array_shift( a, iter / 2 );
        end loop;
        ts_end := clock_timestamp();

        RAISE INFO 'Using array_shift for % iterations over % elements = %',
                   iter, array_length( a, iter / 2 ),
                        ( ts_end - ts_begin );

        ts_begin := clock_timestamp();
        for i in 1 .. iter loop
                perform shift_plperl( a, iter / 2 );
        end loop;
        ts_end := clock_timestamp();

        RAISE INFO 'Using array_shift for % iterations over % elements = %',
                   iter, array_length( a, 1 ),
                        ( ts_end - ts_begin );

END
$CODE$;
```
<br/>
<br/>

where changing the `iter` variable makes the code to run more shifts against the same array.

In the following table, I show some results made on the same tiny crappy virtual machine. Please consider that the function used are:
- `shift` a PL/PgSQL iteration based approach where, at each iteration the leftmost element of the array is removed;
- `shiftx` a PL/PgSQL approach that slices the array;
- `array_shift` is the PL/PgSQL function that executes the single query proposed by Stefan Stefanov;
- `shift_plperl` a PL/Perl function that exploits the `shift` Perl operator.

<br/>
<br/>

{:class="table table-bordered"}
|------------+--------+------------------------+------------------+------------------+------------------|
| Iterations | shifts | `shift`                | `shiftx`         | `array_shift`    | `shift_plperl`   |
|:----------:+:------:+-----------------------:+-----------------:+-----------------:+-----------------:|
| 2000       | 1000  | `23.03905` secs     	   | `00.007952` secs | `00.518305` secs | `00.604843` secs |
| 5000       | 2500  | `05:44.236687` mins 	   | `00.020937` secs | `03.039885` secs | `03.672881` secs |
| 7000       | 3500  | `15:23.3999` mins   	   | `00.033396` secs | `05.97662` secs  | `07.132485` secs |
| 8000       | 4000  | `23:02.73513` mins  	   | `00.044517` secs | `07.445447` secs | `10.236968` secs |
| 10000      | 5000  | `44:46.496029` mins     | `00.048962` secs | `12.211704` secs | `15.091066` secs |
| 12000      | 6000  | `01:16:53.758594` hours | `00.060169` secs | `17.198828` secs | `20.911864` secs |
|============+========+========================+==================+==================+==================|

<br/>
<br/>

Long story short: the PL/PgSQL iteration based approach (`shift`) is by far the slowest approach, while the array-slice approach (`shiftx`) is the fastest one. The PL/Perl and query-only approach are comparable, with the latter being a little faster than the former probably due to PL/Perl requiring to marshall the arguments in and out of the function.

<br/>
Clearly, the above is not a complete benchmarking, and has not been executed multiple times to get average results. However, the above does suffice in providing an idea of how the different approaches relate to each other.


## Where is the SQL based solution spending its time?

It's interesting to try to understand where **Stefan Stefanov**'s solution is spending most of its execution time, and `EXPLAIN` comes to a rescue here.


<br/>
<br/>
```sql
testdb=> explain analyze with 
   arr as ( select array_agg( v ) v from generate_series( 1, 100000 ) v )
   ,arr_tbl(el, arr_index) as (
       select u.* from unnest( (select * from arr) ) with ordinality u 
   )
   select (select el from arr_tbl where arr_index = 5000), 
          (select array_agg(el order by arr_index) 
                          from arr_tbl where arr_index > 5000);
						  
                                                                    QUERY PLAN                                                                      
-----------------------------------------------------------------------------------------------------------------------------------------------------
 Result  (cost=1250.60..1250.61 rows=1 width=36) (actual time=105.725..105.727 rows=1 loops=1)
   CTE arr_tbl
     ->  Function Scan on unnest u  (cost=1250.03..1250.13 rows=10 width=12) (actual time=59.127..66.255 rows=100000 loops=1)
           InitPlan 1 (returns $0)
             ->  Aggregate  (cost=1250.01..1250.02 rows=1 width=32) (actual time=51.840..51.841 rows=1 loops=1)
                   ->  Function Scan on generate_series v  (cost=0.00..1000.00 rows=100000 width=4) (actual time=30.879..40.827 rows=100000 loops=1)
   InitPlan 3 (returns $2)
     ->  CTE Scan on arr_tbl  (cost=0.00..0.22 rows=1 width=4) (actual time=60.106..79.291 rows=1 loops=1)
           Filter: (arr_index = 5000)
           Rows Removed by Filter: 99999
   InitPlan 4 (returns $3)
     ->  Aggregate  (cost=0.23..0.24 rows=1 width=32) (actual time=26.330..26.331 rows=1 loops=1)
	   ->  CTE Scan on arr_tbl arr_tbl_1  (cost=0.00..0.22 rows=3 width=12) (actual time=0.270..7.439 rows=95000 loops=1)
                 Filter: (arr_index > 5000)
                 Rows Removed by Filter: 5000
 Planning Time: 0.323 ms
 Execution Time: 106.301 ms

						  
```
<br/>
<br/>

The main node where there is time consuption is the `CTE Scan` to find out the `head`: it consumes more than `20` milliseconds. That node is produced by the first main subquery, and it requires a scan of the materialized CTE.
Clearly I'm not considering the time consumed to produce the array, i.e., the `InitPlan 1`, because it is used only to feed the query.


# Conclusions

While it is easy enough to implement a shift-like operation for PostgreSQL arrays, either by PL/PgSQL or a nested query, performances will never met the PostgreSQL array slicing.
