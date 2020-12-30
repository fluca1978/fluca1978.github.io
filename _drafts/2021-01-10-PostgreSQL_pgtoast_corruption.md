---
layout: post
title:  "PostgreSQL TOAST Data Corruption (ERROR:  unexpected chunk number)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---


PostgreSQL TOAST Data Corruption (ERROR:  unexpected chunk number)
---
**T**he **O**versize **A**ttribute **S**torage **T**ecnique (TOAST) is a way that allows PostgreSQL to store any kind of attribute within the table.
<br/>
PostgreSQL stores data into data pages that have a fixed size, usually `8 kB`; this means there is no room for a variable content (e.g., a string) that grows more than a single data page. To solve the problem, PostgreSQL uses TOAST: when an attribute value is too large to be stored in the table data page, PostgreSQL **transparently** moves the content to an external storage, namely `pg_toast`, where the content is split into *chunks* (parts) and stored as a set ot chunk tuples.
When you ask back your content, PostgreSQL transparently seeks the chunks, recompose them in the right order, and provide the result to you. It is like the system executes a transparent join between your main table and the `pg_toast` one.

<br/>
<br/>
Unluckily sometime the TOAST storage can be damage, by accident often, resulting in data corruption. The problem is that such corruption goes often *unseen* until the real content is required: in other words your table looks fine unless you select that exact content that has been stored *off-line* into TOAST.
<br/>
<br/>
In this article I introduce a couple of functions that can serve as a basis to find out damaged TOAST data.
<br/>
I've written such functions to do exactly the above job: help me identify the records that have been damaged, so that I can decide how to restore them (and here you should insert any backup good advice as you wish).
<br/>
<br/>
This article is divided into two parts:
- the first one creates an examples and damaged it by purpose, so that you can try the code;
- the second part shows how to use the functions and get some results.


<br/>
<br/>
The code of the functions can be found online [on my Gitlab repository](https://gitlab.com/fluca1978/fluca1978-pg-utils/-/blob/master/examples/toast/find_bad_toast.sql){:target="_blank"}. As usual, any comment and improvement is appreciated.
<br/>
Inspiration for this technique comes [from Josh Berkus excellent article](http://www.databasesoup.com/2013/10/de-corrupting-toast-tables.html{:target="_blank"}.

# Create an example: corrupt your TOAST data

Assume we create the following table within our database:

<br/>
<br/>
```sql
testdb=> create table example_toast( a int, b text, c float, d varchar(10000) );

testdb=> alter table example_toast add column pk serial primary key;

testdb=> insert into example_toast
select x, repeat( 'fluca1978', x * 8000 ), x * 1.2, 
          repeat( 'fluca1978', x * 10 )
from generate_series( 1, 210 ) x;

testdb=> select pg_size_pretty( pg_relation_size( 'example_toast' ) );
 pg_size_pretty 
----------------
 8192 bytes
(1 row)

```
<br/>
<br/>

The table has been filled with four different types of data, each initialized with a different value, with particular regard to the text types that have been initialized to *long* contents.
The table results in a very small one, and occupies exactly one data page.
<br/>
**Does the table has any TOAST-ed data?** We can check that `reltoastrelid` has a value:


<br/>
<br/>
```sql
testdb=> select relname, relfilenode, reltoastrelid from pg_class where relkind = 'r' and oid = 'example_toast'::regclass;
    relname    | relfilenode | reltoastrelid 
---------------|-------------|---------------
 example_toast |       52367 |         44178
(1 row)
```
<br/>
<br/>

Therefore, the table has the `44178` TOAST table associated.

## It's time for a corruption!

In order to make the toasted data faulty, we can use an old Perl script of mine that is going to insert a crappy string into a data file. The script is really simple, as you can see:

<br/>
<br/>
```perl
#!env perl

open my $db_file, "+<", $ARGV[ 0 ]
     || die "Cannot open data file!\n\n";
seek $db_file, ( 8 * 1024 ) + $ARGV[ 1 ], 0;

print { $db_file } "Hello Corrupted Database!";
close $db_file;
```
<br/>
<br/>


Having placed a corruption script, we need to find out the data file that must be corrupted: it is the TOAST table we are going to damage, and we can get the path to a disk file using the PostgreSQL functions.


<br/>
<br/>
```sql
testdb=> select relname, relfilenode, reltoastrelid, 
                pg_relation_filepath( reltoastrelid ) 
                from pg_class where relkind = 'r' 
                and oid = 'example_toast'::regclass;
                
-[ RECORD 1 ]--------|-----------------
relname              | example_toast
relfilenode          | 52367
reltoastrelid        | 44175
pg_relation_filepath | base/24815/52368

```
<br/>
<br/>

We can now corrupt the data on the datafile `base/24815/44175`:


<br/>
<br/>
```shell
% sudo -u postgres \ 
        perl /usr/local/bin/do_corruption.pl \
             /postgres/12/base/24815/44175   \
             12345
```
<br/>
<br/>

**WARNING: don't try this at home**, or better, do try against a test-only database!


## Find out the corruption

What happens if we query the table now? Well, we asked for a data corruption and we got it!


<br/>
<br/>
```sql

testdb=> \o test.txt
testdb=> select b,d from example_toast;
ERROR:  unexpected chunk number 1126199148 (expected 2) for toast value 60699 in pg_toast_44175
```
<br/>
<br/>

Please note that I sent the otuput of the query to a file to make the whole buffer fill the blog post.



# Searching for the error: find out corrupted TOAST data

The data on the TOAST storage has been damaged, and **it is now required to find out which tuples have been affected by the damage** so that you can decide the right strategy for recovery of that data.
<br/>
I have built a couple of functions that can help you find out the damaged tuples. Let's see the final result and then allow me to discuss the details:

<br/>
<br/>
```sql
testdb=> select * from f_find_bad_toast( 'example_toast', 'pk' );


-[ RECORD 1 ]----|-------------------------------------------------------------
total            | 210
ok               | 207
ko               | 3
health_ratio     | 98.57142857142857
damage_ratio     | 1.4285714285714286
description      | Table example_toast has 1.4285714285714286% toast 
                   data damaged (toast relation pg_toast.pg_toast_44175 
                   on disk file [base/24815/85136])
damage_tuple_ids | {110,111,112}

```
<br/>
<br/>

We now have a report that tells us that 3 records out of the 210 total ones have been damaged: the record ranging from `pk` 110 to 112 are the ones hitted by the data corruption, and therefore the toast data is wrong. The good news is that 98% of our table is healthy.

<br/>
<br/>
The function `f_find_bad_toast` accepts the table name and a column that must be unique (and therefore acting as a surrogate primary key). The function inspects every single record in the table and tries to de-toast its data. The final result is that, in this example, every single tuple has been corrupted.


The function `f_find_bad_toast` does the following:
1) performs a few sanity checks, and *gets the list of TOASTable attributes of the table*;
2) prepare an SQL query `SELECT` to query every single toastable attribute;
3) converts the toasted column into `text` and performs a few aggregate operations on that data, so to force the *detoasting*;
4) if an exception arises, the function stores the primary key of the tuple to indicate that there is an error on such tuple.

<br/>
Internally, the function exploits another custom piece of code, `f_enumerate_toastable_columns` that provides a list of those columns that could have been stored on TOAST.
As an example:


<br/>
<br/>
```sql
testdb=> select * from f_enumerate_toastable_columns( 'example_toast' );
 f_enumerate_toastable_columns 
-------------------------------
 b
 d
(2 rows)
```
<br/>
<br/>

As you can see, only columns with variable length could be stored in the TOAST area.

## How `f_find_bad_toast()` works

You can get some hints on the internal working behavior by increasing the debug message level:


<br/>
<br/>
```sql
testdb=> set client_min_messages to debug;

testdb=> select * from f_find_bad_toast( 'example_toast', 'pk' );
...
DEBUG:  Preparing to de-toast record pk = 161
DEBUG:  Prepared query [SELECT  lower( b::text )  ||  lower( d::text )  FROM example_toast WHERE pk = '161']
DEBUG:  Succesfully executed query [SELECT  lower( b::text )  ||  lower( d::text )  FROM example_toast WHERE pk = '161']
,,,

```
<br/>
<br/>

As you can see, the function inspects **every single record at a time** (i.e., it can be really slow on large tables!) and builds an appropriate query to de-toast toastable data. Then the query is executed, the result is placed into a variable and the length of the result is computed; if this succeed the data has been detoasted, otherwise there was a problem reading the toasted data.
In short, the function does:


<br/>
<br/>
```sql
BEGIN
   EXECUTE query_detoast
   INTO    current_detoasted_data;

  PERFORM  length( current_detoasted_data );
  RAISE DEBUG 'Succesfully executed query [%]', query_detoast;
  ok_counter = ok_counter + 1;
EXCEPTION
  WHEN OTHERS THEN
       ko_counter = ko_counter + 1;
       wrong_tuple_ids = array_append( wrong_tuple_ids, current_pk );
       RAISE NOTICE 'Record with % = % of table % has corrupted toast data!', pk, current_pk, tablez;

END;

```
<br/>
<br/>

The `query_detoast` is built for every single record, as for instance `SELECT  lower( b::text )  ||  lower( d::text )  FROM example_toast WHERE pk = '161'`.


### Arguments

The `f_find_bad_toast` function accepts four arguments:
- the table name;
- the surrogate primary key column name;
- an optional limit clause, useful when inspecting very large tables;
- an optional offset argument, useful when iterating over the same table.

A possible improvement could be to automatically find out a table's primary key, for example by inspecting the system catalogs.


### `f_enumerate_toastable_columns`

The `f_enumerate_toastable_columns` inspects the system catalogs to find out which attributes can be stored by TOAST. At its core, it returns every item in `pg_attribute` that has a storage of `x` (extended) or `e` (external), meaning that the attribute has been stored outside of the main table.



# Conclusions

The TOAST mechanism is great, but until you detoast the content of your data you could not notice a problem in it.
Periodically run tools based on the above functions can help you determine if a problem has been generated, and so far I've only experienced human-caused damages, so don't worry about your PostgreSQL cluster as far as nobody disturbs it!
