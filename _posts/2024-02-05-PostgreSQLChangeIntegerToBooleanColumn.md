---
layout: post
title:  "Changing a Column from Integer to Boolean in One Transaction"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A way to fix some oddity that comes from other databases.

# Changing a Column from Integer to Boolean in One Transaction

I was migrating a database from SQLite3 to PostgreSQL, not because the former isn't good, rather because the latter shines!

SQLite3 does not have booleans, so the tricky way to simulate booleans is to use integer columns (or characters, or whatever works for you), and I was in this situation with a table `cassification` having a `miscellaneous` column with only two values: `1` to indicate `true` and `0` to indicate `false`. Moreover, the column had a default value set to `0` (i.e., `false`).

While this is not a problem, it is really annoying when doing queries and data manipulation.
Luckily PostgreSQL allows us for a quick fix of the column, migrating its data type to another. Unluckily, there is no straighforward evaluation of an integer into a boolean, so PostgreSQL is not able to understand how to migrate values, but it is quite simple to instrument it to follow the right path.

First of all, there is the need to check the original column values to identify if, by accident, some not-boolean-ish values have been stored. This is really simple, since you can do something like:

<br/>
<br/>
```sql
testdb=> SELECT count(*), miscellaneous
         FROM classification
		 WHERE miscellaneous NOT IN ( 0, 1 )
		 GROUP BY miscellaneous;
```
<br/>
<br/>

Now it is time to migrate the column.

**PostgreSQL allows for transactional DDL statements**, that means you can run multiple DDL statements within a transaction.
Therefore, within a single transaction, it is possible to:
- drop the column default value;
- change the column data type, telling PostgreSQL about how to migrate the data;
- assign a new default value to the new column.

Moreover, PostgreSQL is able to execute a single `ALTER TABLE` with multiple `ALTER COLUMN` statements, something that reminds me Oracle's `ALTER TABLE MODIFY ( )` expression:

<br/>
<br/>
```sql
testdb=> alter table classification
            alter column miscellaneous drop default,
            alter column miscellaneous set data type boolean
                 using
                 case miscellaneous when 1 then true else false end,
           alter column miscellaneous set default false;
```
<br/>
<br/>

Done!

The first `alter column` statement removes the default value, the second one uses a `case` to convert an integer into a boolean, and the last one adds a default value.

A more verbose way of doing the same thing is:

<br/>
<br/>
```sql
testdb=> BEGIN;
testdb=> alter table classification alter column miscellaneous drop default;
testdb=> alter table classification alter column miscellaneous set data type boolean
                 using
                 case miscellaneous when 1 then true else false end;

testdb=> alter table classification  alter column miscellaneous set default false;
testdb=> COMMIT;

```
<br/>
<br/>`


The [documentation and examples for ALTER TABLE](https://www.postgresql.org/docs/16/sql-altertable.html){:target="_blank"} provide more details about how to change the data type in similar situations.
