---
layout: post
title:  "PostgreSQL & FreeBSD: using the rc(8) to configure another instance"
author: Luca Ferrari
tags:
- PostgreSQL
- FreeBSD
permalink: /:year/:month/:day/:title.html
---
The rc(8) startup system is a clean and powerful way to manage services, but if you need to test a secondary PostgreSQL instance
on a FreeBSD machine, having to deal with ~pg_ctl~ can become annoying. Here there is a simple and dirty trick to
configure a secondary PostgreSQL service(8).

## PostgreSQL & FreeBSD: using the rc(8) to configure another instance
-----
When dealing with multiple PostgreSQL instances on the same machine you often have a situation where one instance is a kind of *master*, i.e., it is managed by the `service(8)` facility, while the others need some extra care with command line and `pg_ctl`.

In this post I show how to quickly get a new service to handle a secondary instance of PostgreSQL via `service(8)` in a real transparent way.

**WARNING: this a dirty way of doing things, and if you mess up with variables you risk to damage your main instance, so please test it
carefully and be sure to understand script shells, variables, and `rc(8)`!**

In a *plain* installation, the main instance managed by `service(8)` is named, guess what, `postgresql`, so that in order to start (or restart or whatever) you can do the following:

```shell
% sudo service postgresql start
```

Now, imagine the secondary instance you want to deal with will be named **`postgresql_replica`** (yes, underscores are admitted!).
In order to be able to do the following:

```shell
% sudo service postgresql_replica start
```

you have to follow these steps:

1. configure the `postgresql_replica` rc variabiles in `/etc/rc.conf`, something like the following could suffice:
```shell
postgresql_replica_enable="YES"
postgresql_replica_data="/mnt/data3/pgdata"
```
**Be very careful about the data directory, that has not to be the same of the main instance!**

2. **copy** the `/usr/local/etc/rc.d/postgresql` file into `/usr/local/etc/rc.d/postgresql_replica` one. Please note, do not link the two, hard copy the latter because it requires changes that are incompatible with the main instance.

3. edit the `/usr/local/etc/rc.d/postgresql_replica` so that each variable is evaluated with the `postgresql_replica_*` rc variables.
For instance:

```shell
command=/usr/local/bin/pg_ctl

. /etc/rc.subr

load_rc_config postgresql_replica

postgresql_enable=${postgresql_replica_enable:-"NO"}
postgresql_flags=${postgresql_replica_flags:-"-w -s -m fast"}
postgresql_user=${postgresql_replica_user:-"postgres"}
eval postgresql_data=${postgresql_replica_data:-"~${postgresql_user}/data96"}
postgresql_class=${postgresql_replica_class:-"default"}
postgresql_initdb_flags=${postgresql_replica_initdb_flags:-"--encoding=utf-8 --lc-collate=C"}

name=postgresql_replica
rcvar=postgresql_replica_enable
extra_commands="reload initdb"
```

As you can see, all the righthand variables changed from `postgresql` to `postgresql_replica`, as well as the configuration loaded that is now the same name `postgresql_replica`. This does suffice to start the `postgresql_replica` service and do the other common stuff.
