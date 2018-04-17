---
layout: post
title:  "What happens when you add a column?"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Adding a column with a default value requires a full table rewrite, and therefore it is often suggested to avoid the default value. However, it is possible to add the default value without having PostgreSQL perform the full table rewrite.

# Adding a column via `ALTER TABLE ADD COLUMN`

In order to demonstrate what does PostgreSQL when a new column is added, consider the following simple table to begin with:

```sql
> CREATE TABLE foo( i int );
> INSERT INTO foo(i) 
  SELECT v 
  FROM generate_series(1, 100000) v;
```

Suppose it is required to add a column, *with a default value*, to the table. When an `ALTER TABLE ADD COLUMN` is issued, and a default value is provided, PostgreSQL performs a **full update** of the whole table, that is all the 100k tuples are updated immediatly:

```sql
 > ALTER TABLE foo ADD COLUMN c char(1) DEFAULT 'A';

ALTER TABLE
Time: 180.997 ms

> SELECT distinct( c ) FROM foo;
 c 
---
 A
(1 row)
```

As readers can see, the `c` column has been added and all the tuples have been updated to the default value.

When the number of tuples is really high, performing such `ADD COLUMN` will result in a very huge database activity. Therefore, it is often suggested to perform the `ADD COLUMN` without a default value, so to get the column added very fast, and then issuing the update of the column value.

This is of course something with a different meaning: the default value is not placed in the table and therefore it is possible to insert some `NULL` values in such column.

There is a little trick to avoid the above problem when it is required a default value: issue two different `ALTER TABLE` to (1) add the column and (2) set the default value:

```sql
> ALTER TABLE foo ADD COLUMN v char(1);
ALTER TABLE
Time: 0.564 ms

> ALTER TABLE foo ALTER COLUMN v SET DEFAULT 'B';
ALTER TABLE
Time: 0.724 ms

> SELECT distinct( v ) FROM foo;
 v 
---
 
(1 row)

```

As readers can see, this time the column has a default value **but the table has not been rewritten** (and therefore the addition of the column is almost immediate preserving the semantic meaning of the column itself). It is now possible to perform the update of the table in a batch whenever possible or appropriate, while not incurring into the problem of risking wrong tuples to hit the table.
