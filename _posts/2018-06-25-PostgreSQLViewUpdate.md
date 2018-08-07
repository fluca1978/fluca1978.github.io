---
layout: post
title:  "PostgreSQL: cannot alter type of a column used by a view or rule"
author: Luca Ferrari
tags:
- postgresql
- itpug
permalink: /:year/:month/:day/:title.html
---
How to cascade changes to a view over a table?


# PostgreSQL: cannot alter type of a column used by a view or rule

In a lectur on PostgreSQL a participant came up with a puzzling question: how to cascade an `ALTER COLUMN` from a table to a view? There are two main approaches: using the power of DDL transactionable commands or alter the system catalog. In this post I explain how to do both.

<br/>
<br/>
Imagine you have a table and a view (either dynamic or materialized) built on top of it:

```sql
> CREATE TABLE t( pk smallint, t char(2) );
> CREATE VIEW vw_t AS SELECT * FROM t;
```

and of course, both the view and the table reflect the same field structure

```sql
> \d t
                    Table "public.t"
 Column |     Type     | Collation | Nullable | Default 
--------|--------------|-----------|----------|---------
 pk     | smallint     |           |          | 
 t      | character(2) |           |          | 

> \d vw_t
                   View "public.vw_t"
 Column |     Type     | Collation | Nullable | Default 
--------|--------------|-----------|----------|---------
 pk     | smallint     |           |          | 
 t      | character(2) |           |          | 
```

What happens if the `t` table changes the structure? PostgreSQL does not allow this since there is a dependency from `vw_t` over `t`:

```sql
> ALTER TABLE t ALTER COLUMN t TYPE char(10);
ERROR:  cannot alter type of a column used by a view or rule
DETAIL:  rule _RETURN on view vw_t depends on column "t"

> ALTER TABLE t ALTER COLUMN pk TYPE bigint;
ERROR:  cannot alter type of a column used by a view or rule
DETAIL:  rule _RETURN on view vw_t depends on column "pk"
```

The `DETAIL` message provides an hint on the problem: the `"_RETURN"` rule is the special way PostgreSQL handles views: it does bounce the `SELECT** statement to the underlying table.

**There is only a correct approach to solve the problem: since PostgreSQL allows any DDL command to be included in a transaction, a transaction can be used to drop the view, alter the table and recreate the view**:

```sql
> BEGIN;
> DROP VIEW vw_t;
> ALTER TABLE t ALTER COLUMN pk TYPE bigint;
> ALTER TABLE t ALTER COLUMN t TYPE char(10);
> CREATE VIEW vw_t AS SELECT * FROM t;
> COMMIT;
```


Since views do not need to be compiled, as in other database systems, there should be never the case to not execute the above on a live system. In the very rare case where there is no possibility to start the above transaction, the system catalog can be changed to reflect the wanted structure.
Inspecting the `pg_attribute` system catalog shows how the table has been defined:

```sql
> SELECT attname, atttypmod, attlen 
  FROM pg_attribute
  WHERE attname IN ( 'pk', 't' )
  AND attrelid = 't'::regclass;
  
 attname | atttypmod | attlen 
---------|-----------|--------
 pk      |        -1 |      2
 t       |         6 |     -1
```

In short, the `pk` column being a number has no type modifiers (`atttypmod`) and a total length (`attlen`) of `2` bytes (it is a `smallint`); on the other hand, the `t` column has no specific length but a modifier of `6`, in particular `4` added by the length itself (in this case, being a `char(2)` is `2 + 4 = 6`).

<br/>
<br/>

Let's say the table must to be changed so that the `pk` becomes a `bigint` and the `t` column a `char(10)`: it is possible to force it on the catalog itself. While it is possible to modify the `t` column directly on the catalog increasing the type modifier length, this is not possible on the `pk` column because the `pg_attribute.attlen` is not a real value rather a copy of `pg_type.typlen` as shown [in the `pg_attribute` documentation](https://www.postgresql.org/docs/10/static/catalog-pg-attribute.html).
Therefore, the update of the `t` table has to be done as follows:
1) change the type of the numeric column `pk`;
2) change the type modifier length of the column `t`, adding to the desired value 4 bytes required by PostgreSQL bookeping.
<br/>
The above two steps must be performed as a superuser, so **if you cannot gain superuser privileges, you cannot update the table structure via the system catalog**. It is also important to note that the names in `pg_type` are not SQL names, but PostgreSQL internal names; in other words the `bigint` type is named `int8`.
<br/>
Let's see the procedure in action:

```sql
# BEGIN; 
# UPDATE pg_attribute 
  SET atttypid = 
   ( SELECT oid 
     FROM pg_type 
     WHERE typname = 'int8' ) 
  WHERE attname = 'pk'
  AND attrelid = 't'::regclass;

# UPDATE pg_attribute SET atttypmod = 14 
  WHERE attname = 't'
  AND attrelid = 't'::regclass;

# \d t
                    Table "public.t"
 Column |     Type      | Collation | Nullable | Default 
--------|---------------|-----------|----------|---------
 pk     | bigint        |           |          | 
 t      | character(10) |           |          | 

# \d vw_t
                   View "public.vw_t"
 Column |     Type     | Collation | Nullable | Default 
--------|--------------|-----------|----------|---------
 pk     | smallint     |           |          | 
 t      | character(2) |           |          | 

-- if ready commit changes ...
```


Of course, this is not what is required, so I strongly discourage to commit the above transaction. There is the need to replay the very same statements against the whole dependent objects (in this case `vw_t`). If the view remains at its old structure odd behaviors can arise, since extracted values can be regularly read (i.e., the `t` column will be limited from the underlying table, so a `char(10)`) but updates of the view can result in data truncation.
<br/>
**Again, in PostgreSQL the correct way to perform such structure change is using an ordinary transaction, dropping the dependent objects and recreating them within the same transaction once the table has changed.**
<br/>

Take extreme care when working with the system catalog, and ensure you have a valid backup of your data before changing it, since this is not the intended way to let PostgreSQL protect your data!

