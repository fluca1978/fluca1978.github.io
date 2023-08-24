---
layout: post
title:  "Using custom variables as per-session global variables"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A possible trick to emulate per-session global variables.

# Using custom variables as per-session global variables

In a thread [in the italian mailing list](https://www.freelists.org/post/postgresql-it/Alternativa-a-variabil-globali-di-sessione,8){:target="_blank"} we were discussing about *session global variables*, something I believe is a bad idea, no matter what is the problem you are trying to solve, but probably a more *database-oriented* approach could solve it (e.g., temporary tables).

<br/>

One thing I did not know, and I discovered thanks to the above discussion (credits to *Andrea Adami*) is that PostgreSQL allows the definition of custom variables by means of `SET`. Well, `SET` is of course the way to configure a GUC, that is a configuration parameter of the cluster.
As you probably know, all GUCs that have a name without a namespace are *cluster-wide*, while those with a prefix belong to an extension.

<br/>

Since PostgreSQL does not know in advance if an extension has been loaded or not, and since extension can be loaded at run-time, the cluster allows the user to set parameters that contain a prefix in the name. Documentation [can be found here](https://www.postgresql.org/docs/15/runtime-config-custom.html){:target="_blank"}. Therefore, it is possible to use `SET` to define a *fake* GUC variable to be used in queries and functions.

<br/>

As an example:

<br/>
<br/>
```sql
testdb=> SET fluca1978.favourite_database TO 'PostgreSQL';
SET
testdb=> SHOW fluca1978.favourite_database;
 fluca1978.favourite_database
------------------------------
 PostgreSQL
(1 row)
testdb=> SELECT 'Luca loves ' || current_setting( 'fluca1978.favourite_database' );
       ?column?
-----------------------
 Luca loves PostgreSQL
(1 row)

```
<br/>
<br/>

The variable behaves as a `user` context parameter, and honor also transaction boundaries:

<br/>
<br/>
```sql
estdb=> SELECT 'Luca loves ' || current_setting( 'fluca1978.favourite_database' );
       ?column?
-----------------------
 Luca loves PostgreSQL
(1 row)

testdb=> BEGIN;
BEGIN
testdb=*> SET fluca1978.favourite_database TO 'Oracle';
SET
testdb=*> SELECT 'Luca loves ' || current_setting( 'fluca1978.favourite_database' );
     ?column?
-------------------
 Luca loves Oracle
(1 row)

-- argh!
-- rollback!

testdb=*> ROLLBACK;
ROLLBACK
testdb=> SELECT 'Luca loves ' || current_setting( 'fluca1978.favourite_database' );
       ?column?
-----------------------
 Luca loves PostgreSQL
(1 row)

```
<br/>
<br/>


Clearly, this kind of variable is **session-scoped** and cannot be shared among different sessions:

<br/>
<br/>
```sql
testdb=> SELECT pg_backend_pid(), current_setting( 'fluca1978.favourite_database' );
 pg_backend_pid | current_setting
----------------+-----------------
            857 | PostgreSQL
(1 row)


-- in another session

testdb=> SELECT pg_backend_pid(), current_setting( 'fluca1978.favourite_database' );
ERROR:  unrecognized configuration parameter "fluca1978.favourite_database"


```
<br/>
<br/>


# Conclusions

I don't recommend this usage of *dynamic session-scoped variables*, since a temporary table is usually a better idea and provides pretty much the same solution. It is however interesting to know that PostgreSQL has this behavior. Clearly, it is important to avoid clashes in variable names (a thing that you don't risk with temporary tables) against really existing GUCs defined by an extension.
