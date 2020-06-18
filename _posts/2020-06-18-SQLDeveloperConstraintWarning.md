---
layout: post
title:  "ORA-2449 and the Constraint Dependencies"
author: Luca Ferrari
tags:
- oracle
- planet-postgresql-org
- java
permalink: /:year/:month/:day/:title.html
---
What happens if you try to drop a table that is referenced by another table?

# ORA-2449 and the Constraint Dependencies

Oracle clients seems somehow a little goofy when you have to deal with dependencies.
<br/>
Imagine you have two tables, `a` that references table `b`; you can generate the tables as follows:

```sql
SQL> CREATE TABLE a( pk int PRIMARY KEY );

Table created.

SQL> CREATE TABLE b( pk int PRIMARY KEY );

Table created.

SQL> ALTER TABLE a ADD( b_ref int REFERENCES b(pk) );

Table altered.

SQL> COMMIT;

Commit complete.
```


<br/>
As you can see, the tables are empty, there is no effective data but there is a clear reference made by the foreign key `b_ref` that connects table `a` to table `b`.
<br/>
So far, so good!
<br/>
Now, let's try to delete table `b`, on which `a` depends on:


```sql
SQL> DROP TABLE b;
DROP TABLE b
           *
ERROR at line 1:
ORA-02449: unique/primary keys in table referenced by foreign keys
```

<br/>
Great! Oracle, as we are expecting, is telling us that we cannot drop the refenced table unless we remove the dependency from the dependent object.
<br/>
However, please note how **Oracle is not telling us what dependency is preventing us from dropping the table**!
<br/>
The situation is pretty much the same if you execute the `SQL Developer` client:

<br/>
<br/>
<center>
<img src="/images/posts/oracle/oracle_constraint_warning.png" />
</center>
<br/>
<br/>

Please note how the `SQL Developer` warning dialog is even suggesting us to execute a query against the Oracle catalogs to see which constraints are making the `DROP` fail. Not only, Oracle `SQL Developer` is so lazy to not even complete the query for us: instead of placing the table name in the query and presenting us a *copy-and-paste* ready statement, it tells us to execute something like `SELECT ... WHERE TABLE_NAME = 'tabname'`.
<br/>
I'm pretty sure someone at least one time has executed a query searching a table named *`tabname`*!
<BR/>
<BR/>
**Why is not Oracle giving us an hint about the constraints?**
<br/>
<BR/>
Being used to PostgreSQL, I can say that this should be correct behavior. In fact, if you try this in PostgreSQL you get a clear warning about which constraint is preventing you to delete the table:


```sql
testdb=> CREATE TABLE a( pk serial primary key );
CREATE TABLE
testdb=> CREATE TABLE b( pk serial primary key );
CREATE TABLE
testdb=> alter table a add b_ref int references b(pk);
ALTER TABLE
testdb=> drop table b;
ERROR:  cannot drop table b because other objects depend on it
DETAIL:  constraint a_b_ref_fkey on table a depends on table b
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

<br/>
In particular, the message `constraint a_b_ref_fkey on table a depends on table b` gives us a really clear explaination of what we should search for to fix the "problem".
<br/>
Not only, PostgreSQL is reminding us that, if we want to quickly get rid of the table, we can use the `DROP...CASCADE` statement to force PostgreSQL to take action.
