---
layout: post
title:  "Usage of disk space in Oracle and PostgreSQL: a simple use case"
author: Luca Ferrari
tags:
- postgresql
- oracle
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A very non-scientific comparison about the two database engines.

# Usage of disk space in Oracle and PostgreSQL

A few days ago I built a table in Oracle (11, if that matters) to store a few hundred megabytes of data.
But I don't feel at home using Oracle, so I decided to export the data and import it back in PostgreSQL 12.
<br/>
Surprisingly, PostgreSQL requires more data space to store the same amount of data.
<br/>
<br/>
**I'm not saying anything about who is the best, and I don't know the exact reasons why this happens, however this is just what I've observed hpoing this can be useful to someone else!**
<br/>
*So please don't flame!*
<br/>

## Table structure

The table is really simple, and holds data about files on a disk. It does not have even a key, since it is just data I must mangle and then throw away.

```sql
testdb=> \d my_schema.my_files
                        Table "my_schema.my_files"
  Column   |          Type           | Collation | Nullable | Default 
-----------|-------------------------|-----------|----------|---------
 filename  | character varying(200)  |           |          | 
 directory | character varying(2048) |           |          | 
 md5sum    | character varying(128)  |           |          | 
 bytes     | bigint                  |           |          | 

```

I've seen no changes in using `text` against a `varchar`, I used the latter just to be as similar as possible in the definition with Oracle.
<br/>
*The table is populated with `1872529` tuples (around 2 million tuples).*

## Oracle Disk Space

Oracle requires `312 MB` to store the data:

```sql
 select segment_name,sum(bytes)/1024/1024 MB
    , count(segment_name)
    , blocks * 8192 / (1024 * 1024 )
    from user_segments
    where segment_type='TABLE'
    and segment_name=upper('MY_FILES')
    group by segment_name, blocks ;
```

The results of the above query are:
- `312 MB` of data;
- `39936` blocks, that are something similar to PostgreSQL data pages.

The table has `110` extents, but I'm not sure how they account in the space compuation.

## PostgreSQL Disk Space

The same data in PostgreSQL required `324 MB`, so `12 MB` more than Oracle, that is **roughly 4% more of disk space**. It is therefore possible to say that the overall space is pretty much the same:


```sql
testdb=> SELECT reltuples, relpages, 
         pg_size_pretty( pg_relation_size( 'my_schema.my_files' ) ) 
         FROM pg_class WHERE relname = 'my_files' AND relkind = 'r';
         
  reltuples   | relpages | pg_size_pretty 
--------------|----------|----------------
 1.872529e+06 |    41491 | 324 MB
(1 row)
```

Please note that `fillfactor` has been set to 100% and the table has been `VACUUM`ed.


## Counting Pages

What I can see, is that PostgreSQL uses `41491` data pages, while Oracle uses `39936`, so `1555` less data pages. Again, that is roughly the same 4% we already saw on effective space, that lead me think the Oracle datapages have the same size as PostgreSQL.
<br/>
In fact, asking for the datapage size:

```sql
SQL> show parameter db_block_size;

NAME          TYPE    VALUE 
------------- ------- ----- 
db_block_size integer 8192 
``

shows the same size as PostgreSQL.

# Conclusions

**I really don't have any. I know too little about Oracle storage to say why there is this difference in size, and I'm sure this is neither an advantage of Oracle nor a drawback of PostgreSQL. **
<br/>
I don't even know if this is the default behavior for any use-case, I hardly think so, but it is interesting to know that even a simple use-case like this can require a little more space on disk.
