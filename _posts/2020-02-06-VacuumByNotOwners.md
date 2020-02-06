---
layout: post
title:  "Executing VACUUM by non-owner user"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
VACUUM needs to be run by the object owner!

# Executing VACUUM by non-owner user

The [documentation about `VACUUM` clearly states it](https://www.postgresql.org/docs/12/sql-vacuum.html){:target="_blank"}:

         To vacuum a table, one must ordinarily be the table's owner or a superuser. 
         However, database owners are allowed to vacuum all tables in their databases, 
         except shared catalogs. 
         [...]
         VACUUM cannot be executed inside a transaction block.

There is not an ACL flag about `VACUUM`, that means **you cannot `GRANT` someone else to execute `VACUUM`**.
<br/>
Period.
<Br/>
<br/>
Therefore there is no escape: **in order to run `VACUUM` you must to be either (i) the *object owner* or (ii) *the database owner* or,as you can imagine, (iii) one of the cluster superuser(s).**
<br/>
<br/>
Why am I insisting on this? Because some friends of mine argued that it is always possible to escape restrictions with functions an `SECURITY DEFINER` options. In this particular case, one could think to define a function that executes `VACUUM`, then apply the `SECURITY DEFINER` option so that the function will run as the object owner, and then provide (i.e., `GRANT`) execution permission to a normal user.
<br/>
*WRONG!*
<br/>
The fact that `VACUUM` cannot be executed within a transaction block means you cannot use such an approach, because a function is executed within a transaction block.
<br/>
And if now you are asking yourself why `VACUUM` cannot be wrapped in a transaction block, just explain me how to `ROLLBACK` a `VACUUM` execution, it will be an interesting and fantasyland explaination!

<br/>
So, what is going to happen if you define a `VACUUM`-function?
Let's quickly see what the database does:

```sql
CREATE OR REPLACE FUNCTION
do_vacuum( t text )
RETURNS VOID
AS $$
BEGIN
  EXECUTE 'VACUUM FULL VERBOSE '
  || quote_ident( t );
END
$$
LANGUAGE plpgsql;
```

This will not work, since **`VACUUM` cannot be invoked by a function** (have I already written this?):

```sql
testdb=> select do_vacuum( 'foo' );
ERROR:  VACUUM cannot be executed from a function
CONTEXT:  SQL statement "VACUUM FULL VERBOSE foo"
PL/pgSQL function do_vacuum(text) line 3 at EXECUTE
```

Changing the function into a procedure does not solve the problem, because **`VACUUM` cannot be invoked by a function** (have I already written this?):


```sql
testdb=> CREATE OR REPLACE PROCEDURE
 do_vacuum( t text )
 AS $$
 BEGIN
   EXECUTE 'VACUUM FULL VERBOSE '
   || quote_ident( t );
 END
 $$
 LANGUAGE plpgsql;
CREATE PROCEDURE

testdb=> CALL do_vacuum( 'foo' );
ERROR:  VACUUM cannot be executed from a function
CONTEXT:  SQL statement "VACUUM FULL VERBOSE foo"
PL/pgSQL function do_vacuum(text) line 3 at EXECUTE
```


# Conclusions

**`VACUUM` cannot be wrapped in a transaction nor a routine**, therefore in order to execute it you must be a "special" user, with special simply meaning the owner, or the database owner, or a superuser.
