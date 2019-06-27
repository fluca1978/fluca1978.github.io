---
layout: post
title:  "PostgreSQL Administrator Account WITH NOLOGIN (recover your role)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
Today I got an email from a friend of mine that locked out of his own database due to a little mistake.

# PostgreSQL Administrator Account WITH NOLOGIN (recover your `postgres` role)

What if you get locked out your own cluster due to a simple and, to some extent, stupid error?
Let's see it in quick list of steps.
<br/>
First of all, lock the default `postgres` account so that the default administrator cannot any more log in the clsuter:

```shell
% psql -U postgres -c "ALTER ROLE postgres WITH NOLOGIN" testdb
ALTER ROLE

% psql -U postgres -c "SELECT version();" testdb               
psql: FATAL:  role "postgres" is not permitted to log in
```

*What a mess!*
<br/>
<br/>
PostgreSQL has a specific recovery mode, called **single user mode**, that resemble the operating system single user mode and can be used for such situations. Let's see how.
<br/>
First of all, **shut down the cluster**, avoid more damages of what you have already done!

```shell
% sudo service postgresql stop
```

<br/>
Now, start the `postgres` process in single user mode. You need to know the data directory of your cluster in order for it to work:

```shell
% sudo -u postgres postgres --single -D /mnt/pg_data/pgdata/11.1

PostgreSQL stand-alone backend 11.3
backend> 
```

What happened? I used the operating system user `postgres` to launch the operating system process `postgres` (ok there's a little name confusion here!) in single (`--single`) mode for my own data directory (`-D`). I got a prompt, I'm connected to the backend process directly, so this is not the same as a local or TCP/IP connection: I'm interacting with the backend process itself. Luckily, the backend process can speak SQL! Therefore, I can *reset* my administrator role:

```sql
backend> ALTER ROLE postgres WITH LOGIN;
backend> 
```

Please note that, while the backend process can speak SQL, it does not speak the same way `psql` does: there is no need for a semicolon and an `<enter>` will send the statement to the backend. Anyway, I can now release the backend process as I would do with any other operating system process, gently or not, for instance via `CTRL-D` (*End of File*).

```shell
backend>  CTRL-D
%
```

It is now time to restart the cluster and check if the user `postgres` can connect again:

```shell
% sudo service postgresql start
% psql -U postgres -c "SELECT CURRENT_DATE;" testdb
 current_date 
--------------
 2019-06-27
(1 row)
```

The world is an happy place again!
