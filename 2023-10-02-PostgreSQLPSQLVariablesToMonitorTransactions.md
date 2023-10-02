---
layout: post
title:  "Using psql Variables to Introspect Your Script"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A little trick to monitor your own running transaction in term of time and data size.

# Using psql Variables to Introspect Your Script

`psql` is by far my favourite SQL text client, it has features that even the most expensive database tools provide.
One very interesting property of `psql` is to support internal variables, pretty much like the variables you can find in a shell.

Since I often find myself doing some queries to get information about a transaction, in term of time and quantity of data manipulated, and doing manually the math, I decided that `psql` can do this for me by means of variables.


## The Use Case: Quantitative Data About a Transaction

I want to run a *long* transaction that does some data manipulation and transformation, and I want to get an idea about how much it is going to `cost` me such a transaction, so that I can estimate how to apply the same transformation in production.

Usually, I begin the transaction having a look at the current time and WAL position, and I do the same at the end of the transaction.
Doing the difference between the values provides me an *hint* about the wall clock time and the amount of data (assuming no other activity is going on the database).
As an example:

<br/>
<br/>
```sql

testdb=> BEGIN;
BEGIN
testdb=*> SELECT clock_timestamp() AS begin_clock
testdb-*> , pg_current_wal_lsn() AS begin_lsn;
         begin_clock          | begin_lsn
------------------------------+------------
 2023-09-29 10:32:05.51654+02 | 2/A39CC3C0
(1 row)

testdb=*> INSERT INTO t( t )
testdb-*> SELECT 'Dummy ' || v
testdb-*> FROM generate_series( 1, 1000000 ) v;
INSERT 0 1000000
testdb=*> SELECT clock_timestamp() AS end_clock
, pg_current_wal_lsn() AS end_lsn;
           end_clock           |  end_lsn
-------------------------------+------------
 2023-09-29 10:32:48.511892+02 | 2/A81AC000
(1 row)

testdb=*> COMMIT;

```
<br/>
<br/>

Now that I have the times and WAL lsn positions, I can *manually* compute the *cost* of this transaction by copying and pasting the results:


<br/>
<br/>
```sql
testdb=> SELECT '2023-09-29 10:32:48.511892+02'::timestamp
         - '2023-09-29 10:32:05.51654+02'::timestamp AS wall_clock
        , pg_size_pretty( pg_wal_lsn_diff( '2/A81AC000', '2/A39CC3C0' ) ) as size;
   wall_clock    | size
-----------------+-------
 00:00:42.995352 | 72 MB

```
<br/>
<br/>

So the transaction took `42` seconds and produced around `72 MB` of data (in the WALs).
Note that I had to manually copy and paste every single value in order for the query to compute the difference I want.



## Using `psql` variables to obtain the computation automatically

If I store the *begin* and *end* values into `psql` variables, I can use an immutable query to compute the same results, without having to copy and paste the single values.

This trick is made possible by the special command `\gset`, that allows for the declaration and definition of variables out of a query result.


<br/>
<br/>
```sql
testdb=> BEGIN;
BEGIN

testdb=*> SELECT clock_timestamp() AS clock
, pg_current_wal_lsn() AS lsn \gset begin_

testdb=*> INSERT INTO t( t )
SELECT 'Dummy ' || v
FROM generate_series( 1, 1000000 ) v;
INSERT 0 1000000

testdb=*> SELECT clock_timestamp() AS clock
, pg_current_wal_lsn() AS lsn \gset end_

testdb=*> SELECT :'end_clock'::timestamp - :'begin_clock'::timestamp as wall_clock
, pg_size_pretty( pg_wal_lsn_diff( :'end_lsn', :'begin_lsn' ) ) as size;
   wall_clock    | size
-----------------+-------
 00:00:11.400421 | 72 MB


testdb=*> COMMIT;
COMMIT
```
<br/>
<br/>


The two query to get the timing and WAL lsn informations are similar, and exploit a `gset begin_` and `\gset end_` command respectively. The first command takes the output of the query and, for each column, creates a variable with the given prefix (`begin_`) and the column name, therefore `begin_clock` and `begin_lsn`. The second query does the very same with the prefix `end_`, therefore creating `end_clock` and `end_lsn` variables.

The interesting part is the last query, that by now is totally automated and performs the differences between `end_` and `start_` values (please note the quoting and casting). Thanks to this little trick, I can now place such queries at the boundaries of my scripts and get as output the result I want or need to monitor the transaction.

Clearly, this approach can be extended, so you can have variables to track the number of tuples, the number of tables created or deleted, and so on. The key idea is to have a kind of *catch-all* set of queries that depend on variables you will define systematically in your scripts.


### Why is the second transacction faster than the first one?

In the above example I shown two identical transactions, but the first one is slower, in terms of execution time, than the second one.
The answer is simple: in the first transaction I was literally typing in the SQL statements, while in the second I was recalling them from the `psql` history. It is only a matter of typing the statements!



# Conclusions

When I do professional training and present the `psql` command line client I see disappointment in my trainee faces. However, the more I go on explaining how flexible and powerful `psql` is, the more the classroom likes it.
Thanks to the capabiliy of automagically set variables from a query output, `psql` allows you to automate some tasks including your own script introspection.
