---
layout: post
title:  "PostgreSQL 15: changes in the public schema permissions"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
The upcoming new release of PostgreSQL does some changes on the `public` schema permissions.

# PostgreSQL 15: changes in the `public` schema permissions

In PostgreSQL 15 the default `public` schema that every database has will have a different set of permissions. In fact, before PostgreSQL 15, every user could manipulate the `public` schema of a database he is not owner.
Since the upcoming new version, only the database owner will be granted full access to the `public` schema, while other users will need to get an explicit `GRANT`:

<br/>
Imagine the user `luca` is owner of the database `testdb`: it means he can do whatever he wants on the database.

<br/>
<br/>

``` sql
testdb=> SHOW server_version;
 server_version
----------------
 15beta2
(1 row)

testdb=> SELECT current_role, current_user;
 current_role | current_user
--------------+--------------
 luca         | luca
(1 row)

testdb=> CREATE TABLE mytable( t text );
CREATE TABLE

```
<br/>
<br/>

On the other hand, another user, let's say `pgbench`, cannot:

<br/>
<br/>

``` sql
testdb=> SELECT current_role, current_user;
 current_role | current_user
--------------+--------------
 pgbench      | pgbench
(1 row)

testdb=> CREATE TABLE mytable2( t text );
ERROR:  permission denied for schema public
LINE 1: CREATE TABLE mytable2( t text );

testdb=> select * from mytable;
ERROR:  permission denied for table mytable
```
<br/>
<br/>

That means that `public` is not managed as a user defined schema, and therefore in order to allow other users to do operations, an explicit `GRANT` must be executed.
<br/>
What has changed is that **there is no more the `CREATE` permission on `public` schema**, while `YUSAGE` is as before. Therefore, in order to allow not-owners to create objects, an explicit `GRANT CREATE ON SCHEMA public TO pgbench` statement myust be executed.
<br/>
This affects newly created databases, not those restored from previous backups.
<br/>
But there is a trick that could help in setting back the previous behavior: if you set the permissions on the `template1` (or in a template database) you could have them for free on new databases:


<br/>
<br/>

``` sql
template1=# GRANT CREATE ON SCHEMA public TO PUBLIC;
GRANT

template1=# CREATE DATABASE newdb WITH OWNER luca;
CREATE DATABASE
```
<br/>
<br/>

And now, collecting as not-owning user:

<br/>
<br/>

``` sql
% psql -U pgbench -h localhost newdb

newdb=> create table foo( i int );
CREATE TABLE

```
<br7>
<br/>

the permissions are as in previous PostgreSQL versions.
<br/>
It is not clear if the above trick will remain in place once the PostgreSQL version exists the beta status, in any case I discourage you to adopt it. The choice of revoking by default privileges on the `public` schmea could be annoying, but is a good choice in term of security and forces you to decide how to deal with permissions.
