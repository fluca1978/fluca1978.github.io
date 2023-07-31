---
layout: post
title:  "PostgreSQL Cluster Connection Limits"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A brief look to understand how the main cluster connection limits work.

# PostgreSQL Cluster Connection Limits

PostgreSQL has two main connection limit tunables that allow the system administrator to decide what is the maximum number of connections the cluster will support and, in case an emergency activity has to be performed, what part of such connections is reserved to superusers.


PostgreSQL 16 is going to introduce a new parameter named `reserved_connections` among the other two `max_connections` and `superuser_reserved_connections`:

<br/>
<br/>
```shell
% psql -U postgres -h localhost -c 'SHOW SERVER_VERSION;'
 server_version
----------------
 16beta2
(1 row)

% psql -U postgres -h localhost -c "SELECT name, setting FROM pg_settings WHERE name like '%connections' and name not like 'log%'; "
              name              | setting
--------------------------------+---------
 max_connections                | 100
 reserved_connections           | 0
 superuser_reserved_connections | 3

```
<br/>
<br/>

The above are the default settings, that have not been changed since several releases of PostgreSQL.

The idea is to allow a fine grain tuning of how connections will be limited depending on the user asking for it.
In this article, I try to briefly explain the difference between the two main settings (`max_connections` and `superuser_reserved_connections`) and the freshly introduced one (`reserved_user_connections`).



## The Connection Limits Settings: `max_connections` and `superuser_reserved_connections`

First of all, the main idea is that the cluster **is going to accept no more connections than `max_connections`**, hence `100` in the above. Among the `max_connections` available, `superuser_reserved_connections` will be kept empty for incoming connections from superuser roles.
<br/>
In other words, clients and application will be able to establish `max_connections - superuser_reserved_connections` connections.
<br/>
<br/>
It is simple enough to demonstrate this by means of `pgbench`:

<br/>
<br/>
```shell
% pgbench -U pgbench -T 60 -P 5 -n -c 100 -h localhost pgbench
pgbench (15.3, server 16beta2)
pgbench: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  remaining connection slots are reserved for roles with SUPERUSER
pgbench: error: could not create connection for client 97

```
<br/>
<br/>

In the above, I asked `pgbench` to create `100` concurrent connections, that is the `max_connections` value. That fails because three connections are reserved to superusers.

<br/>
It is possible to demonstrate this using `pgbench` and simultaneously opening other connections. In a terminal launch the following:

<br/>
<br/>
```shell
% pgbench -U pgbench -T 120 -P 5 -n -c 97 -h localhost pgbench
```
<br/>
<br/>

that will consume all available user-level connections and will last for two minutes. Meanwhile, in another terminal, if you try to login as a non-superuser you get an error, while superuser can connect:

<br/>
<br/>
```shell
% psql -U pgbench -h localhost pgbench
psql: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  remaining connection slots are reserved for roles with SUPERUSER

% psql -U postgres -h localhost pgbench
psql (15.3, server 16beta2)
WARNING: psql major version 15, server major version 16.
         Some psql features might not work.
Type "help" for help.

pgbench=#

```
<br/>
<br/>



## The New Connection Limit `reserved_connections`

As already written, this is a new parameter introduced by PostgreSQL 16.
This parameter allows connections by user granted by the `pg_use_reserved_connections`, and is a way to make some non-superuser role more powerful, granting to him more capabilities.


<br/>
First of all, let's set the parameter to `10` connections; please note that being a network related parameter it is required a reboot of the cluster.

<br/>
<br/>
```shell
% psql -h localhost -U postgres -c 'ALTER SYSTEM SET reserved_connections TO 10;'
ALTER SYSTEM

% pgenv stop
% pgenv start
```
<br/>
<br/>

In the above I use `pgenv` as my PostgreSQL manager, but that is not the important part.
After that, there is the need to grant some user(s) with the `pg_use_reserved_connections` permission:

<br/>
<br/>
```shell
% psql -U postgres -h localhost -c 'GRANT pg_use_reserved_connections TO luca;'
GRANT ROLE

```
<br/>
<br/>



It is now time to try:

<br/>
<br/>
```shell
% pgbench -U pgbench -T 120 -P 5 -n -c 97 -h localhost pgbench
pgbench (15.3, server 16beta2)
pgbench: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  remaining connection slots are reserved for roles with privileges of the "pg_use_reserved_connections" role
pgbench: error: could not create connection for client 87

```
<br/>
<br/>

As you can see, `pgbench` is no more able to obtain up to `97` connections because now `10` are reserved for non-superuser roles with the `pg_use_reserved_connections`. Therefore, the only way to make it work is to low the concurrent connections to `max_connections - reserved_connections - superuser_reserved_connections`, that means `100 - 10 - 3 = 87`.


<br/>
<br/>
```shell
% pgbench -U pgbench -T 120 -P 5 -n -c 87 -h localhost pgbench
```
<br/>
<br/>

and while the above is working, you can try to connect from another concurrent session:

<br/>
<br/>
```shell
% psql -U pgbench -h localhost pgbench
psql: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  remaining connection slots are reserved for roles with privileges of the "pg_use_reserved_connections" role

% psql -U luca -h localhost pgbench
psql (15.3, server 16beta2)
WARNING: psql major version 15, server major version 16.
         Some psql features might not work.
Type "help" for help.

pgbench=>

```
<br/>
<br/>

In the first attempt, the connection fails because the `pgbench` user does not have any more connection slots to use, or better, there are no connection slots within the cluster to use.
<br/>
However, the user `luca` succeed at connecting because he has the special `pg_use_reserved_connections` permission and there are still available slots.

<br/>
It is important to note that **no matter if your cluster does not have any role with the `pg_use_reserved_connections`, once the setting `reserved_connections` is not zero the cluster will keep such connection slots available**!
In other words, use `reserved_connections` only when you are sure you are going to grant the permission to a few roles.



# Conclusions

PostgreSQL is able to prevent the system administrator to lock out the cluster, even when the number of connections is approaching the maximum allowance.
Thanks to the new parameter `reserved_connections` added in upcoming PostgreSQL 16, it will be possible to fine-grain tune the connection allowance even better!
