---
layout: post
title:  "Why Dropping a Column does not Reclaim Disk Space? (or better, why is it so fast?)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
You may have noticed how dropping a column is fast in PostgreSQL, haven't you?

# Why Dropping a Column does not Reclaim Disk Space? (or better, why is it so fast?)

Simple answer: **because PostgreSQL knows how to do its job at best**!
<br/>
<br/>

Let's create a dummy table to test this behavior against:

```sql
testdb=> CREATE TABLE foo( i int );
CREATE TABLE

testdb=> INSERT INTO foo 
         SELECT generate_series( 1, 10000000 );
INSERT 0 10000000

testdb=> SELECT pg_size_pretty( pg_relation_size( 'foo' ) );
 pg_size_pretty 
----------------
 346 MB
(1 row)

```

Now, let's add a quite large column to the table and measure how much time does it takes:

```sql
testdb=> \timing
Timing is on.

testdb=> ALTER TABLE foo 
         ADD COLUMN t text 
         DEFAULT md5( random()::text );
ALTER TABLE
Time: 30702,872 ms (00:30,703)
```

What happened? In nearly `31 secs` the table has grown with random data on every row to the extent of `651 MB` (almost the double of the original size):

```sql
testdb=> SELECT pg_size_pretty( pg_relation_size( 'foo' ) );
 pg_size_pretty 
----------------
 651 MB
(1 row)
```

What does PostgreSQL thinks about the columns in this table? Let's query the `pg_attribute` catalog on all those attributes that are user-defined (i.e., `attnum` is a positive value) and inspect the `attisdropped` value that indicates if the column belongs or not to the table:

```sql
testdb=> SELECT attnum, attname, attisdropped 
         FROM pg_attribute a 
         JOIN pg_class c ON c.oid = a.attrelid 
         WHERE c.relname = 'foo' 
               AND c.relkind = 'r' 
               AND a.attnum > 0;
               
 attnum | attname | attisdropped 
--------|---------|--------------
      1 | i       | f
      2 | t       | f
(2 rows)
```

As you can see, both the columns `foo.i` and `foo.t` are valid in the table, that means *they have not been dropped*.

<br/>
<br/>
It is now time to drop the columns and see the results:

```sql
testdb=> ALTER TABLE foo DROP COLUMN t;
ALTER TABLE
Time: 20,237 ms
```

*Pretty impressive, isn't it?*
<br/>
We waited almost `31` seconds to add the new data and no one (`20` *milliseconds*) to drop it away?
<br/>
The [documentation helps understanding it](https://www.postgresql.org/docs/12/sql-altertable.html){:target="_blank"}:

    The DROP COLUMN form does not physically remove the column, but simply makes it invisible to SQL operations. Subsequent insert and update operations in the table will store a null value for the column. Thus, dropping a column is quick but it will not immediately reduce the on-disk size of your table, as the space occupied by the dropped column is not reclaimed. The space will be reclaimed over time as existing rows are updated.
    
There is no reason to immediatly force a table rewrite, the `DROP COLUMN` **invalidates the column** so that is has disappeared *logically* but not *physically*. Let's inspect the table and its attributes again:

```sql
testdb=> SELECT pg_size_pretty( pg_relation_size( 'foo' ) );
 pg_size_pretty 
----------------
 651 MB
(1 row)


testdb=> SELECT attnum, attname, attisdropped 
         FROM pg_attribute a 
         JOIN pg_class c ON c.oid = a.attrelid 
         WHERE c.relname = 'foo' 
               AND c.relkind = 'r' 
               AND a.attnum > 0;
               
 attnum |           attname            | attisdropped 
--------|------------------------------|--------------
      1 | i                            | f
      2 | ........pg.dropped.2........ | t
(2 rows)

```

The table size remained the same, but the `t` attribute has been renamed as `........pg.dropped.2........` and **is now marked as dropped from the table (`attisdropped = t`)**.
<br/>
Does that mean that it is possible from SQL to query the dropped column? No, *this is not a recycle bin* like mechanism:

```sql
testdb=> SELECT i, "........pg.dropped.2........" FROM foo limit 10;
ERROR:  column "........pg.dropped.2........" does not exist
LINE 1: SELECT i, "........pg.dropped.2........" FROM foo limit 10;
```

However, many of the properties of the column data type, such its length, are still in there into `pg_attribute` to allow the system to mangle that column even if the data type itself disappears.

<br/>
Last, let's fire a full table rewrite, for example with a `VACUUM`:

```sql
testdb=> VACUUM FULL foo;
VACUUM
Time: 20231,232 ms (00:20,231)

testdb=> SELECT pg_size_pretty( pg_relation_size( 'foo' ) ); pg_size_pretty 
----------------
 346 MB
(1 row)

Time: 1,519 ms
testdb=> SELECT attnum, attname, attisdropped 
         FROM pg_attribute a 
         JOIN pg_class c ON c.oid = a.attrelid 
         WHERE c.relname = 'foo' 
               AND c.relkind = 'r' 
               AND a.attnum > 0;
               
 attnum |           attname            | attisdropped 
--------|------------------------------|--------------
      1 | i                            | f
      2 | ........pg.dropped.2........ | t
(2 rows)
```

According to the time spent in `VACUUM` something good must be happened, and in fact the table space was reduced to the right (or better, the original) amount of space. 
<br/>
But why the dropped column is still mentioned in `pg_attribute`?
<br/>
In this particular case it would have been dropped quite easily also from `pg_attribute`, but imagine a more complex tble where you drop a column in the middle of the attribute list: PostgreSQL would also have to rewrite all the attribute ordering with a quite expensive amount of work.
<br/>
However, this approach has a potential drawback: being dropped attributes mentioned in `pg_attribute` as *normal* ones, they do count as table attributes and therefore could lower the number of *real active* attributes you can have in the table.



# Conclusions

PostgreSQL way of dropping column is really fast because it involves a catalog update. But that also means disk space is not reclaimed, so in order to do that you need to trigger a full table rewrite.
