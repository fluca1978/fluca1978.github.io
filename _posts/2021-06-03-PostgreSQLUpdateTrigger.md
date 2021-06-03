---
layout: post
title:  "PostgreSQL Builtin Trigger Function to Speed Up Updates"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Did you know PostgreSQL ships with a pre-built trigger function that can speed up UPDATES?

# PostgreSQL Builtin Trigger Function to Speed Up Updates

PostgreSQL ships with an *internal* trigger function, named `suppress_redundant_updates_trigger` that can be used to avoid idempotent updates on a table.
<br/>
The [online documentation](https://www.postgresql.org/docs/12/functions-trigger.html){:target="_blank"} explains very well how to use it, including the fact that the trigger should be fire as last in a trigger chain, and so the trigger name should be alphabetically the last one in natural sorting.
<br/>
But is it worth using such function?
<br/>
Let's find out wth a very trivial example on well known `pgbench` database. First of all, let's consider the initial setup:

<br/>
<br/>
```sql
pgbench=> SELECT count(*), 
          pg_size_pretty( pg_relation_size( 'pgbench_accounts' ) ) 
          FROM pgbench_accounts;
  count   | pg_size_pretty 
----------|----------------
 10000000 | 1281 MB
(1 row)

```
<br/>
<br/>

Now, let's execute an idempotet `UPDATE`, that is something that does not change anything, and monitor the timing:


<br/>
<br/>
```sql
pgbench=> \timing
Timing is on.
pgbench=> UPDATE pgbench_accounts SET filler = filler;
UPDATE 10000000
Time: 307939,763 ms (05:07,940)

pgbench=> SELECT pg_size_pretty( pg_relation_size( 'pgbench_accounts' ) );
 pg_size_pretty 
----------------
 2561 MB
(1 row)

Time: 180,732 ms


```
<br/>
<br/>

Note how the table has doubled its size: this is because of bloating caused by every row being substituted by an exact copy of it.
<br/>
Now, let's create the trigger using the `suppress_redundant_updates_trigger` function, and let's run the same update again, but after a server restart to clean up also the memory.

<br/>
<br/>
```sql
pgbench=> CREATE TRIGGER tr_avoid_idempotent_updates
BEFORE UPDATE ON pgbench_accounts
FOR EACH ROW
EXECUTE FUNCTION suppress_redundant_updates_trigger();

-- restart the server

pgbench=> \timing
Timing is on.
pgbench=> UPDATE pgbench_accounts SET filler = filler;
UPDATE 0
Time: 287588,607 ms (04:47,589)

pgbench=> SELECT pg_size_pretty( pg_relation_size( 'pgbench_accounts' ) );
 pg_size_pretty 
----------------
 2561 MB
(1 row)

```
<br/>
<br/>

The total gain was about `20 secs`, that is a speed up of roughly `7%`, that is not too much at all.
<br/>
However, note how the `UPDATE` reports **zero tuples have been touched**, therefore while the speed up gain is not really exciting, the bloating of the table remains the same as before the `UPDATE` itself.

<br/>
After a full vacuum, the speed up results a lot more, but this can be a counter effect of having in memory already some pages:


<br/>
<br/>
```sql
pgbench=> VACUUM FULL pgbench_accounts ;
VACUUM
Time: 222455,150 ms (03:42,455)
pgbench=> UPDATE pgbench_accounts SET filler = filler;
UPDATE 0
Time: 198104,981 ms (03:18,105)

```
<br/>
<br/>

However, even after a reboot of the server, the time remains lower:


<br/>
<br/>
```sql
pgbench=> UPDATE pgbench_accounts SET filler = filler;
UPDATE 0
Time: 184217,260 ms (03:04,217
```
<br/>
<br/>

So the gain on a *not bloated* table is around `67%` which is much more interesting!


## Timing the trigger execution

How long does it take to execute the trigger function against every row? It is possible to get this information with `EXPLAIN ANALYZE`:

<br/>
<br/>
```sql
pgbench=> EXPLAIN (FORMAT yaml, ANALYZE, VERBOSE, TIMING ) 
          UPDATE pgbench_accounts SET filler = filler;               
                    QUERY PLAN                     
---------------------------------------------------
 - Plan:                                          +
     Node Type: "ModifyTable"                     +
     Operation: "Update"                          +
     Parallel Aware: false                        +
     Relation Name: "pgbench_accounts"            +
     Schema: "public"                             +
     Alias: "pgbench_accounts"                    +
     Startup Cost: 0.00                           +
     Total Cost: 263935.00                        +
     Plan Rows: 10000000                          +
     Plan Width: 103                              +
     Actual Startup Time: 153053.980              +
     Actual Total Time: 153377.845                +
     Actual Rows: 0                               +
     Actual Loops: 1                              +
     Plans:                                       +
       - Node Type: "Seq Scan"                    +
         Parent Relationship: "Member"            +
         Parallel Aware: false                    +
         Relation Name: "pgbench_accounts"        +
         Schema: "public"                         +
         Alias: "pgbench_accounts"                +
         Startup Cost: 0.00                       +
         Total Cost: 263935.00                    +
         Plan Rows: 10000000                      +
         Plan Width: 103                          +
         Actual Startup Time: 8.968               +
         Actual Total Time: 44542.939             +
         Actual Rows: 10000000                    +
         Actual Loops: 1                          +
         Output:                                  +
           - "aid"                                +
           - "bid"                                +
           - "abalance"                           +
           - "filler"                             +
           - "ctid"                               +
   Planning Time: 24.475                          +
   Triggers:                                      +
     - Trigger Name: "tr_avoid_idempotent_updates"+
       Relation: "pgbench_accounts"               +
       Time: 1510.272                             +
       Calls: 10000000                            +
   Execution Time: 159552.624
(1 row)

```
<br/>
<br/>

As you can see, running the trigger requires roughly `1.5 secs` for `10 million` tuples.
<br/>
Assuming the timing is enough accurate and stable, it means `0.00015 msecs` for every tuple, that is not much overhead after all.


<br/>
<br/>
It is possible to provide another table to experiment against, in order to see if the timing for the trigger eecution depends on the data types and its content:


<br/>
<br/>
```sql
pgbench=> create table stuff( pk serial, t text );

pgbench=> INSERT INTO stuff( t ) SELECT repeat( 'abc', 1000 ) 
          from generate_series( 1, 2000000 );
          
          
pgbench=> CREATE TRIGGER tr_avoid_idempotent_updates
BEFORE UPDATE ON stuff 
FOR EACH ROW
EXECUTE FUNCTION suppress_redundant_updates_trigger();


pgbench=> EXPLAIN ( FORMAT yaml, ANALYZE, VERBOSE, TIMING ) 
          UPDATE stuff SET t = t;
          
...
  Triggers:                                      +
     - Trigger Name: "tr_avoid_idempotent_updates"+
       Relation: "stuff"                          +
       Time: 223.227                              +
       Calls: 2000000                             +

```
<br/>
<br/>

Again, the mean execution time of the trigger is `0.00011 msecs`, and very similar (if not equal) results can be obtained with the `pk` column, so I would say that *the execution time of the trigger does not involves the specific type of the column(s) being updated*.


# The Black Behing the Triger Funtion

The `suppress_redundant_updates_trigger` is defined in the file `utils/adt/trigfuncs.c`, and the magic happens in the following piece of code:

<br/>
<br/>
```c
	/* if the tuple payload is the same ... */
	if (newtuple->t_len == oldtuple->t_len &&
		newheader->t_hoff == oldheader->t_hoff &&
		(HeapTupleHeaderGetNatts(newheader) ==
		 HeapTupleHeaderGetNatts(oldheader)) &&
		((newheader->t_infomask & ~HEAP_XACT_MASK) ==
		 (oldheader->t_infomask & ~HEAP_XACT_MASK)) &&
		memcmp(((char *) newheader) + SizeofHeapTupleHeader,
			   ((char *) oldheader) + SizeofHeapTupleHeader,
			   newtuple->t_len - SizeofHeapTupleHeader) == 0)
	{
		/* ... then suppress the update */
		rettuple = NULL;
	}

```
<br/>
<br/>

that essentially compares the old and the new tuple to see if they have the same headers, the same number of attributes, and of course the same content of the memory representation (by means of `memcpm(3)`).


## Doing in `plpgsql`

It is possible to implement a basic function in `plpgsql` by means of the `IS DISTINCT FROM` operator:

<br/>
<br/>
```sql
```
<br/>
<br/>

and the execution with this trigger in place results in:

<br/>
<br/>
```sql
pgbench=> drop trigger tr_avoid_idempotent_updates on pgbench_accounts;
DROP TRIGGER
                                                     
pgbench=> create trigger tr_avoid_idempotent_updates 
before update on pgbench_accounts              
for each row
execute function f_avoid_idempotent_updates();
CREATE TRIGGER


pgbench=> update pgbench_accounts set filler = filler;
UPDATE 0
Time: 167400,098 ms (02:47,400)
```
<br/>
<br/>

and if you track function executions:

<br/>
<br/>
```sql
pgbench=> select * from pg_stat_user_functions ;
-[ RECORD 1 ]--------------------------
funcid     | 36672
schemaname | public
funcname   | f_avoid_idempotent_updates
calls      | 10000000
total_time | 21276.741
self_time  | 21276.741

```
<br/>
<br/>
that indicates that `21 secs` are spent in doing the trigger analysis, so roughly `0,0021 msecs` spent for each tuple. This is by far much more expensive of the C default function (that was roughly `0.00015 msecs`).
<br/>
Similar results are emphasized by the `EXPLAIN ANALYZE` output:

<br/>
<br/>
```sql
pgbench=> EXPLAIN (FORMAT yaml, ANALYZE, TIMING )
          UPDATE pgbench SET filler = filler;
...
  |   Triggers:                                      +
  |     - Trigger Name: "tr_avoid_idempotent_updates"+
  |       Relation: "pgbench_accounts"               +
  |       Time: 23002.383                            +
  |       Calls: 10000000                            +
  |   Execution Time: 163343.183

```
<br/>
<br/>

Here the `Time` is around `23000 msecs` while with the C native function it was about `1500 msecs`.

# Conclusions

The internal `suppress_redundant_updates_trigger` function can be useful for reducing **both time and bloating** against large batches of `UPDATE`s. 
<br/>
The function is implemented in the C language and checks if the memory content of the tuples is the same or not, and this makes this approach really powerful and not so error prone as defining a custom trigger function by the user.
