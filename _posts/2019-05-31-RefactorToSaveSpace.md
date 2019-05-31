---
layout: post
title:  "Normalize to save space"

author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
It is no surprise at all: a normalized database requires less space on disk than a not-normalized one.

# Normalize to save space
Sometimes you get a database that *Just Works* (tm) but its data is not normalized. I'm not a big fan of data normalization, I mean it does surely matter, but I don't tend to "over-normalize" data ahead of design.
However, one of my database was growing more and more because of a table with a few repeated extra information.
<br/>
Of course a normalized database gives you some more disk space at the cost of the joins during query execution, but having a decent server and a small join table is enough to sleep at night!
<br/>
Let's see what we are talking about:

```sql
mydb=# select pg_size_pretty( pg_database_size( 'mydb' ) );
 pg_size_pretty
----------------
 13 GB
(1 row)
```

Ok, `13 GB` is not something scarying, let's say it is a fair database to work on (please note the size if reported *after* a full `VACUUM`).
In such database, I've a table `root` that handles a lot of data from hardware sensors; such table is of course partitioned on a time base scale. One thing the table was storing was information about the sensor *name*, a text string repeated over and over on child tables too. While this was not a problem in the beginning, it was wasting space over time.
<br/>
<br/>
Shame on me!
<br/>
<br/>
Let's go normalize the table!
<br/>
Normalizing a table is quite straightforward, and I'm not interesting in sharing details here. Let's say this was quite easy because **my users where executing query against a view and not the root table**, therefore I simply:
- created a join table;
- populated the join table extracting data from the root table;
- (within a transaction) removed the columns from the root table, modified the view (by dropping and recreating it).

<br/>
How much space was I supposed to gain?
Let's see how much space did the column occupy:

```sql
mydb=# select pg_column_size( 'app.root.sensor_name' );
 pg_column_size
----------------
             20
(1 row)

mydb=# select count(*) from app.root;
   count
-----------
 126224120


mydb=# select pg_size_pretty( 126224120::bigint * 20 );
 pg_size_pretty
----------------
 2408 MB
```



The text column was estimated 20 bytes, that on 126 milion of tuples was around `2,4 GB` of disk space.
After the transaction, I did a `VACUUM FULL` to let PostgreSQL re-arrange the disk space and I got the expected result:

```sql
mydb=# select pg_size_pretty( pg_database_size( 'mydb' ));
 pg_size_pretty 
----------------
 9234 MB
```

Please note that the gained space is a lot more than the one estimated becauce I also refactored other columns here and there. **But the normalized database proved to be less space hungry**. Remember that the starting size was already *vacuumed*, so there is no extra space gain due to dead rows lying around.
<br/>
*All the queries are working, the space is optimized, my users are happy, I'm happy!*


