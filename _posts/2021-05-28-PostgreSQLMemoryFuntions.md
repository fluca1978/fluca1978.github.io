---
layout: post
title:  "Memory inspection thru pg_buffercache"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A tiny set of functions to glance at the memory usage in the PostgreSQL system.

# Memory inspection thru pg_buffercache

`pg_buffercache` is a very useful extension that allows for the inspection of the memory as used by a live PostgreSQL instance. The [extension](https://www.postgresql.org/docs/current/pgbuffercache.html){:target="_blank"} is available by means of the contrib module and is very useful to take a look at the memory usage, in other words the usage of the `shared_buffers`.
<br/>
Thanks to this module it is possible to clearly understand the memory consumption and, therefore, the correct tuning of the `shared_buffers` parameter.
<br/>
A few years ago [I wrote a set of example queries](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/pg_buffercache.sql){:target="_blank"} to interact with the module and get a glance at the memory usage. While those queries were a starting point, they had some issues especially when a table was not consuming memory (disibion by zero, and so on).
<br/>
<br/>
I finally found the time to produce a cleaner approach to those queries, so I [re-implemented all the queries by means of functions](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/memory.sql){:target="_blank"}. The script is a `psql` script, and uses some special backslash commands, but you can extract the SQL pure part and execute it by means of another client.
<br/>
The script creates a `memory` schema and places all the functions into such schema; the functions have a name that starts with `f_memory`, so that they should not clash with existing functions. 
<br/>
In the following I describe every function.
<br/>
Please note that the idea here is to provide a background about memory inspection, there is still room for improvements and fixes!

## Installing the functions

It does suffice to execute the `memory.sql` psql script to get the creation of the schema `memory` and all the functions into such schema. The script provides some information about the objects created:

<br/>
<br/>
```sql
tfdb=# \i memory.sql 
Creating a schema named memory...
All objects created!
Try one of the following functions:
 - memory.f_memory() to get very basic information
 - memory.f_memory_usage() to get information about the whole memory
 - memory.f_memory_usage_by_database() to get information about single databases
 - memory.f_memory_usage_by_table() to get information about tables in the current database
 - memory.f_memory_usage_by_table_cumulative() to get cumulative information for tables

You can add the memory schema to the search path.
Try running the following query while testing the database (e.g., via pgbench):

select memory.f_memory_usage();
\watch 5
```
<br/>
<br/>


## The output of the functions

All the function accept a boolean `human` flag, that by default is set to `true`. If the flag is set the output of the memory dimensions will be formatted using `pg_size_pretty()`, therefore will be in a *human readable format*. Otherwise the output will be formatted as plain number of bytes.

<br/>
<br/>
```sql
tfdb=# select * from memory.f_memory();
 total  |  used  |  free  
--------|--------|--------
 800 MB | 101 MB | 699 MB
(1 row)

tfdb=# select * from memory.f_memory( false );
   total   |   used    |   free    
-----------|-----------|-----------
 838860800 | 106168320 | 732692480
(1 row)
```
<br/>
<br/>


## Utility functions
There are a few utility functions that are used as a backbone to build the others. In particular:
- `memory.f_check_pg_buffercache()` it checks that the extension `pg_buffercache` is installed into the database;
- `memory.f_check_user()` checks that the user is either an administrator or has the privileges to run `pg_buffercache` functions;
- `memory.f_check()` calls the previous two functions and raises an exception if the check fails. This function is invoked by all the other *memory related* functions, so that before the function is run the user can get an alert about missing pieces;
- `memory.f_usagecounter_to_string()` provides a textual description of the `pg_buffercache.usagecount` value;
- `memory.f_tablename()` provides the name of a table, index or view os anything that will appear in the output of other functions;
- `memory.f_print_bytes()` prints the amount of bytes as text, using either `pg_size_pretty()` or plain text conversion. This is used in every function to support the above mentioned `human` flag.


## Available functions

The available functions to inspect the memory usage are described in the following.

### f_memory()
The function `memory.f_memory()` provides a glance at free and used memory in the cluster.

<br/>
<br/>
```sql
tfdb=# select * from memory.f_memory();
 total  |  used  |  free  
--------|--------|--------
 800 MB | 163 MB | 637 MB
(1 row)

```
<br/>
<br/>


### f_memory_usage()

The function `memory.f_memory_usage()` provides a more detailed view about the usage of the memory. In particular it provides the amount of memory used by level of `usagecount`.

<br/>
<br/>
```sql
tfdb=# select * from memory.f_memory_usage();
 total_memory | memory  | percent | cumulative |  description   
--------------|---------|---------|------------|----------------
 800 MB       | 22 MB   | 2.71 %  | 2.71%      | VERY HIGH (5)
 800 MB       | 2536 kB | 0.31 %  | 3.02%      | HIGH (4)
 800 MB       | 1936 kB | 0.24 %  | 3.26%      | MID (3)
 800 MB       | 1888 kB | 0.23 %  | 3.49%      | LOW (2)
 800 MB       | 135 MB  | 16.85 % | 20.34%     | VERY LOW (1)
 800 MB       | 637 MB  | 79.66 % | 100.00%    | == FREE == (0)
(6 rows)
```
<br/>
<br/>

The `memory` column provides the amount of memory used for a specific *region*, and the `percent` columns provide the ratio of memory usage with regard to the total memory. The `cumulative` column provides the amount ratio of the usage level greater than the current one.
<br/>
As an example, in the above there are `135 MB` used not frequently, and thus the `20.34 %` of memory is used from very high to very low.


### f_memory_usage_by_database()

The function `memory.f_memory_usage_by_database()` provides information about the usage of memory by each database in the cluster, and provides also the *caching* amount of every database.


<br/>
<br/>
```sql
pgbench=# select * from memory.f_memory_usage_by_database();
 total_memory |  database   | size_in_memory | size_on_disk | percent_cached | percent_of_memory 
--------------|-------------|----------------|--------------|----------------|-------------------
 256 MB       | pgbench     | 182 MB         | 1505 MB      | 12.11%         | 71.15%
 256 MB       | ltdb        | 608 kB         | 171 MB       | 0.35%          | 0.23%
 256 MB       | postgres    | 544 kB         | 104 MB       | 0.51%          | 0.21%
 256 MB       | restore     | 544 kB         | 104 MB       | 0.51%          | 0.21%
 256 MB       | restore2    | 544 kB         | 104 MB       | 0.51%          | 0.21%
 256 MB       | restore3    | 544 kB         | 104 MB       | 0.51%          | 0.21%
 256 MB       | restore4    | 544 kB         | 8269 kB      | 6.58%          | 0.21%
 256 MB       | template1   | 544 kB         | 8245 kB      | 6.60%          | 0.21%
(8 rows)
         
```
<br/>
<br/>


### f_memory_usage_by_table()

The function `memory.f_memory_usage_by_table()` provides information about the usage of all *tabular like* stuff, in other words about *relations*.


<br/>
<br/>
```sql
tfdb=# select * from memory.f_memory_usage_by_table();
...

 800 MB       | tfdb | (table) respi.y2019m12                         | 8192 bytes | 0.00 %  | VERY HIGH (5)
 800 MB       | tfdb | (table) respi.y2019m12                         | 22 MB      | 2.70 %  | VERY VERY LOW (0)
 800 MB       | tfdb | (index) respi.y2019m12_ts_idx                  | 32 kB      | 0.00 %  | VERY HIGH (5)
 800 MB       | tfdb | (index) respi.y2019m12_ts_idx1                 | 8192 bytes | 0.00 %  | VERY HIGH (5)

```
<br/>
<br/>

### f_memory_usage_by_table_cumulative()

The function `f_memory_usage_by_table_cumulative()` provides an overview of how much memory a single table is "consuming", without any regard to the usage level counter.


<br/>
<br/>
```sql
tfdb=# select * from memory.f_memory_usage_by_table_cumulative();
-[ RECORD 1 ]-----|-----------------------------------------------
total_memory      | 800 MB
database          | tfdb
relation          | (table) respi.y2019m07
memory            | 10 MB
on_disk           | 1159 MB
percent_of_memory | 1.27 %
percent_of_disk   | 0.88%
usagedescription  | any
-[ RECORD 2 ]-----|-----------------------------------------------
total_memory      | 800 MB
database          | tfdb
relation          | (table) respi.y2019m06
memory            | 10 MB
on_disk           | 1156 MB
percent_of_memory | 1.26 %
percent_of_disk   | 0.87%
usagedescription  | any
...
```
<br/>
<br/>

The function accepts the usual `human` argument, but also an integer optional argument that represents the usage counter you are interested in. When specified, the function will show only the amount of memory used with a greater or equal usage counter.

<br/>
<br/>
```sql
tfdb=# select * from memory.f_memory_usage_by_table_cumulative( 5 );
-[ RECORD 1 ]-----|-----------------------------------------------
total_memory      | 800 MB
database          | tfdb
relation          | (table) respi.y2019m07
memory            | 8192 bytes
on_disk           | 1159 MB
percent_of_memory | 0.00 %
percent_of_disk   | 0.00%
usagedescription  | >= VERY HIGH (5)
-[ RECORD 2 ]-----|-----------------------------------------------
total_memory      | 800 MB
database          | tfdb
relation          | (table) respi.y2019m06
memory            | 8192 bytes
on_disk           | 1156 MB
percent_of_memory | 0.00 %
percent_of_disk   | 0.00%
usagedescription  | >= VERY HIGH (5)
...
```
<br/>
<br/>

# Conclusions

The above set of functions can be used as a starting point to build your own set of queries to inspect the memory usage of a live PostgreSQL cluster. There is still room for improvements and reduce the code duplication, so stay tuned for other versions!
