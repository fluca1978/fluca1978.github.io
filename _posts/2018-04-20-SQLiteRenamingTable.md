---
layout: post
title:  "Renaming a table in SQLite3 (and PostgreSQL)"
author: Luca Ferrari
tags:
- sqlite
- postgresql
permalink: /:year/:month/:day/:title.html
---

Renaming a table is a very powerful feature of every database. SQLite3, as flexible as it is, has a feature to not cascade the renaming to foreign keys. Depending on how (and why) you use such feature, it can allow you for doing poweful things or ruin your database.

# Renaming a table: the foreign key problem

When renaming a table via `ALTER TABLE RENAME` what happens to the foreign key that *refer to* the renamed table?
One would expect the foreign key are adjusted too in order to "point" to the *new* table, but that is not usually the case when working with SQLite3. In fact, this database allows for a great flexibility and does not enforce by default very strict data constraint. This is not a drawback, as it allows for instance *quick and dirty* data migration, and moreover it has to be stressed that **SQLite3 can enforce constraints** when required to.

<br/>
<br/>
The problem therefore can be reduced to *foreign key renaming not cascading* to referencing tables. Why such a feature could be useful at all? Well, SQLite3 provides a limited support to `ALTER TABLE`, and in particular it does not provide an implementation of `ALTER TABLE DROP COLUMN`. Therefore, one common way to change a table definition dropping unused columns is to:
1. create a new table with the definition required;
2. copy all data from the original table to the new one (e.g., `INSERT .. SELECT`);
3. drop the original table;
4. rename the new table to the old name.

While performing the above steps, the foreign key could stay *untouched*, therefore pointing always to the *original table* while the latter has (temporarily) disappeared. Once step 4 is performed the foreign key will "point" to the right table; in other words it is like referencing tables did not see any change in the referenced table.

In the following an example of how SQLite3 handles the renaming and foreign key problem, as well as how a much more sofisticated database like PostgreSQL does. This is not meant in any way to criticize any of the two databases, rather it is just a short demonstration of how they handle the problem in different ways.


## Renaming a table: SQLite3

First of all, create a very tiny workbench:

```sql
> CREATE TABLE foo( pk integer PRIMARY KEY AUTOINCREMENT, t text );
> CREATE TABLE bar( pk integer PRIMARY KEY AUTOINCREMENT, foo int, 
                    FOREIGN KEY(foo) REFERENCES foo(pk) );
> INSERT INTO foo( t ) VALUES( 'First row' ), ( 'Second row' );
> INSERT INTO bar( foo ) SELECT pk FROM foo;                    
```

Ensure that the foreign key is in place:

```sql
> .schema bar
CREATE TABLE bar( pk integer PRIMARY KEY AUTOINCREMENT, foo int, 
                  FOREIGN KEY(foo) REFERENCES foo(pk) );
```

Now it is possible to rename the `foo` table:

```sql
> ALTER TABLE foo RENAME TO foo_fighters;
```

and let's see what happened to the referenced table:

```sql
> .schema bar
CREATE TABLE bar( pk integer PRIMARY KEY AUTOINCREMENT, foo int, 
                  FOREIGN KEY(foo) REFERENCES foo(pk) );
```

*Argh!* **The foreign key still references the old table?**
Yes, that is correct. But what about data?

```sql
> SELECT pk FROM foo_fighters;
1
2

-- insert data that does not respect the foreign key
> INSERT INTO bar(foo) VALUES( 5 ), ( 6 );

-- and data is there !
> SELECT * FROM bar;
1|1
2|2
3|5  -- invalid foreign key value 5
4|6  -- invalid foreign key value 6
```

The trick is that **SQLite3 allows for enabling/disabling foreign keys without any regard if they are actually used**. There is in fact a special *pragma*, named `foreign_keys` (no surprise!) that enables or disables foreign keys, as reported in the [official documentation](https://sqlite.org/pragma.html#pragma_foreign_key_list).
On the above database foreign key were disabled:

```sql
> PRAGMA foreign_keys;
0
```

Turning the foreign key check on prevents insertion of wrong tuples, but the existing ones are still there:

```sql
> PRAGMA foreign_keys = ON;
> PRAGMA foreign_keys;
1
> INSERT INTO bar(foo) VALUES( 5 ), ( 6 );
Error: no such table: main.foo
```

Tuples cannot be inserted now because the relation `bar` references another relation `foo` that does no exist any more.
Placing back the `foo` table and trying again to insert the tuples results in the constraint failure:

```sql
> ALTER TABLE foo_fighters RENAME TO foo;
> INSERT INTO bar(foo) VALUES( 5 ), ( 6 );
Error: FOREIGN KEY constraint failed
```

As well, now renaming the table to something else automatically cascades to the foreign key constraint:

```sql
> ALTER TABLE foo RENAME TO foo_fighters;
> .schema bar
CREATE TABLE bar( pk integer PRIMARY KEY AUTOINCREMENT, foo int, 
                  FOREIGN KEY(foo) REFERENCES "foo_fighters"(pk) );
```

## Renaming a table: PostgreSQL

PostgreSQL has a stricter approach to constraints and foreign keys: table renaming cascades to foreign keys in referencing tables. Therefore, in order to allow a *flexible* behavior as the SQLite3 one it is required to:
- drop the foreign key;
- rename the table;
- (optionally) re-create the foreign key.

Since the above has to be performed on every *referencing* table, this is not for free at all!

<br/>
<br/>
In order to demonstrate how PostgreSQL handles table renaming, let's replay the above simple workbench:


```sql
> CREATE TABLE foo( pk serial PRIMARY KEY, t text );
> CREATE TABLE bar( pk serial PRIMARY KEY, foo int, 
                    FOREIGN KEY(foo) REFERENCES foo(pk) );
> INSERT INTO foo( t ) VALUES( 'First row' ), ( 'Second row' );
> INSERT INTO bar( foo ) SELECT pk FROM foo;
```

Let's confirm the foreign key is in place:

```sql
> \d bar
...
Foreign-key constraints:
    "bar_foo_fkey" FOREIGN KEY (foo) REFERENCES foo(pk)

```

And now rename the referenced table:

```sql
> ALTER TABLE foo RENAME TO foo_fighters;
```

and the foreign key has been *aligned* (of course):

```sql
 > \d bar
...
Foreign-key constraints:
    "bar_foo_fkey" FOREIGN KEY (foo) REFERENCES foo_fighters(pk)
```

as well as data is still there:

```sql
> SELECT * 
  FROM foo_fighters f 
   JOIN bar b 
   ON f.pk = b.foo;
   
 pk |     t      | pk | foo 
----|------------|----|-----
  1 | First row  |  1 |   1
  2 | Second row |  2 |   2

```
