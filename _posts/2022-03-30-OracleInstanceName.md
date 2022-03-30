---
layout: post
title:  "Getting the instance name on an Oracle database (and somehow, the PostgreSQL counterpart)"
author: Luca Ferrari
tags:
- oracle
- postgresql
permalink: /:year/:month/:day/:title.html
---
Oracle instance name (and somehow, the PostgreSQL counterpart).

# Getting the instance name on an Oracle database (and somehow, the PostgreSQL counterpart)

In an Oracle database, the connection URI contains a so called *instance name*, a way to discriminate somehow the Oracle engine to which you are connecting to. I am not an Oracle guru (and quite frankly, I don't want to be!), but this is somehow similar to how PostgreSQL databases work (in a bird's eye view): when you connect to a PostgreSQL database you are connecting to an *engine* (a daemon) that is serving the database. When you connect to an Oracle instance, you are connecting to an engine (a daemon) that is serving such engine.
<br/>
A big difference between the two approaches is that Oracle does not provide a *per-databases isolation*, as PostgreSQL does.
<br/>
Let's see an example of URIs, starting from PostgreSQL first:

<br/>
<br/>

``` shell
postgres://luca@miguel/testdb
```

where:
- `postgres` is the protocol;
- `luca` is the username;
` `miguel` is the remote machine;
- `testdb` is the database instance.

<br/>
And here's the Oracle counterpart:

<br/>
<br/>

``` shell
luca[testdb]@miguel/service
```
<br/>
<br/>

There's an added value `service` which is the name of the instance to which you want to connect to.
<br/>
It goes something like this: the Oracle instance is the same as the PostgreSQL daemon serving a `PGDATA`: each one do serve a bunch of *databases* where PostgreSQL managed the latter as separated entities and Oracle does not.

<br/>
<br/>
*Why is it important to be able to discriminate the instance name?*
<br/>
One thing I see very much is that the instance name is used to discriminate between the set of databases for *production* and for *staging*.
<br/>Of course, some other logic can be in place.
<br/>
Luckily, Oracle has the **`v$session`** internal catalog that can be used to get the column (you guess) `instance_name`:


<br/>
<br/>

``` sql
SELECT instance_name
FROM v$instance;
```
<br/>
<br/>


In my Java applications I do something like the following to discriminate between different services:

<br/>
<br/>

``` sql
SELECT
  CASE WHEN instance_name like '%production%' THEN 1
  ELSE  2
  END
 FROM v$instance;
```
<br/>
<br/>

*Be aware that Java will probably manage the return value as a `BigDecimal`!*
<br/>

<br/>
<br/>
What about PostgreSQL? Well, being the internal organization of services and databases different, PostgreSQL does not have anything like the service name. However, there's a couple of configuration parameters that can be used to "simulate" something like the service instance:
- `cluster_name` a string that is used, also, to distinguish between operating system backend processes;
- `update_process_tile` a boolean toggle that is used to update the backend operating system process title.

<br/>
It is possible to use the `cluster_name` value as an indicator of which cluster (i.e., set of databases) you are connected to with a simple query:

<br/>
<br/>

``` sql
SELECT CASE WHEN current_setting( 'cluster_name' ) LIKE '%production%' THEN 1
ELSE 2
END;

```
<br/>
<br/>

that is pretty much the same of the above Oracle query. The idea here is to get the `cluster_name` string out of `current_setting()`, therefore querying PostgreSQL `current_setting()` is pretty much the same as querying Oracle `v$session.instance_name`.
