---
layout: post
title:  "PostgreSQL negative PID"
author: Luca Ferrari
tags:
- postgresql
- itpug
permalink: /:year/:month/:day/:title.html
---
PostgreSQL uses a simple trick to recognize if the server has started in single-user mode.

# PostgreSQL negative PID

First of all, what is /single user mode/? It is a special way to connect directly, as a **single user**, to the PostgreSQL storage, that is the database. Since each conenction is server by a `postgres` process, single user mode means that a single process attached to the current terminal will run. The server will not accept any incoming connection, the user can interact with the server and its storage being sure he is the only user connected to the system.

How can PostgreSQL knows if it is running in single user mode?

Let's start from the bottom: query a server about its status and see if it is in single user mode.

```shell
% sudo -u postgres pg_ctl -D /mnt/data1/pgdata11b3 status
pg_ctl: single-user server is running (PID: 1291)
```

So the above server is running in single user mode, with a PID of 1291. Let's see how PostgreSQL has stored such information into the `postmaster.pid` text file:

```shell
% sudo -u postgres head -n 1 /mnt/data1/pgdata11b3/postmaster.pid
-1291
```

And it's that simple! PostgreSQL stores a negative PID into the pid file in order to inform other tools that the server is running in single user mode. It is quite easy to test it with a normal server start-up, that will produce a regular (positive) PID:

```shell
% sudo -u postgres pg_ctl -D /mnt/data1/pgdata11b3 start 
waiting for server to start....2018-08-13 19:49:11.901 CEST [1335] LOG:  listening on IPv6 address "::1", port 5432
2018-08-13 19:49:11.901 CEST [1335] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2018-08-13 19:49:11.902 CEST [1335] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2018-08-13 19:49:11.910 CEST [1336] LOG:  database system was shut down at 2018-08-13 19:48:59 CEST
2018-08-13 19:49:11.914 CEST [1335] LOG:  database system is ready to accept connections
 done
server started

% sudo -u postgres head -n 1 /mnt/data1/pgdata11b3/postmaster.pid
1335
```
