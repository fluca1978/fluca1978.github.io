---
layout: post
title:  "A glance at doas & pg_ctl"
author: Luca Ferrari
tags:
- freebsd
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A possible system that differs from `sudo`.

# A glance at doas & pg_ctl

`doas(1)` is a replacement for `sudo(1)`, a program that allows you to execute commands as a different user.
The main advantage of using `sudo(1)` and hence `doas(1)` is that you can gain different privileges without the need to know the authentication tokens (e.g., a password) to do that.
<br/>
I use `sudo(1)` on pretty much every machine I use, both Linux and FreeBSD.
<br/>
In this post I glance at `doas(1)` and how it can be quickly configured to run PostgreSQL commands, mainly `pg_ctl`.

## `doas` introduction

`doas(1)` is a program that was born in the [OpenBSD](http://openbsd.org){:target="_blank"} ecosystem as a replacement for `sudo(1)` because, in short, the latter is too big and cannot be easily integrated into the base system.
<br/>
`doas` is now available on FreeBSD and Linux too, so it is worth spending some time to learn how it works.
<br/>
`doas(1)` is based on a configuration file, namely `doas.conf` (in FreeBSD `/usr/local/etc/doas.conf`), that has a syntax a lot clearer than that of `sudo`, at least in my opinion.
<br/>

Rules are pretty simple:
- every line in the configuration file is a rule, and rules are read from top to the bottom;
- a rule can be either `permit` or `deny`, allowing a user to run a command or not;
- a command is prefix by the special keyword `cmd`;
- a target user, that is the user you want to run the command as, is prefix by the keyword `as`;
- the special keyword `nopass` does not ask for password (same as `NOPASSWD` option for `sudo`);
- it is possible to specify or keep the environment or change it.

<br/>
The usage of `doas(1)` is pretty much the same of `sudo(1)`, and mainly;
- `doas` is the entry command;
- `-u` specifies the user to run the command as;
- the command is the remaining part of the command line.

<br/>
<br/>
`doas` has a lot less features (and thus syntax cluttering) than `sudo`, and therefore it is a lot faster and easy to setup, and according to me a lot less prone to errors.

## Using `doas` to control a PostgreSQL cluster

Assuming you want to control a cluster, that is being able to run `pg_ctl` against a cluster, a possible configuration of `doas.conf` is as follows:


<br/>
<br/>
```shell
permit nopass setenv { PGDATA=$PGDATA } luca as postgres cmd  /usr/local/bin/pg_ctl
permit nopass setenv { PGDATA=$PGDATA } luca as postgres cmd  pg_ctl
```
<br/>
<br/>

The two lines are pretty much identical, with the exception that the second allows for a relative path `pg_ctl` command to run. Let's examine the rules:
- `permit nopass` means that the rule allows to do the command without asking for the current user password;
- `luca as postgres` means that the user `luca` to become the user `postgres`, that is allows the current user `luca` to execute a command with the privileges of the local user `postgres`;
- `cmd //usr/local/bin/pg_ctl` specifies which command (both with absolute and relative path) to execute;
- `setenv { PGDATA=$PGDATA }` means that the target user `postgres` will inherit the `PGDATA` variable from the current user `luca`.

<br/>
<br/>
Therefore, it is now possible to issue the following command to stop the cluster:

<br/>
<br/>
```shell
% doas -u postgres pg_ctl stop
waiting for server to shut down.... done
server stopped
```
<br/>
<br/>

That is equivalent to `sudo -u postgres pg_ctl stop` (assuming you have configured `sudo` to keep the environment**.

<br/>
<br/>
**Please note that using `nopass` and _relative paths_ is, in general, a very bad idea. Do not use it in production!**
<br/>
<br/>


Let's execute some other commands:

<br/>
<br/>
```shell
% doas -u postgres initdb /postgres/13
doas: Operation not permitted
```
<br/>
<br/>

Since `doas` does not have any entry for the command `initdb`, it does not allow the user to execute such command. In order to allow the `initdb`, it is possible to add the following lines to `doas.conf`:

<br/>
<br/>
```shell
permit persist setenv { PGDATA=$PGDATA } luca as postgres cmd  /usr/local/bin/initdb
permit persist setenv { PGDATA=$PGDATA } luca as postgres cmd  initdb
```
<br/>
<br/>

and now it is possible to run it:


<br/>
<br/>
```shell
% doas -u postgres initdb /postgres/13
Password:

The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

...
Success. You can now start the database server using:

    pg_ctl -D /postgres/13 -l logfile start

```
<br/>
<br/>

Note how the program asked for a password; this is due to the `persist` authentication mode instead of `nopass`. `persist` is the behaviour that makes `doas(1)` asking for an authentication password and let the user to execute other commands without the same password within a short period of time. Essentially this is the same as the *default* behaviour of `sudo` in most of the default installations.

<br/>
What if the user wants to be able to execute *every* command related to PostgreSQL?
We can configure the user to be able to execute any command as the `postgres` user with a configuration like the following:

<br/>
<br/>
```shell
permit persist setenv { PGDATA=$PGDATA } luca as postgres 
```
<br/>
<br/>

The above allows `luca` to become `postgres` and execute any command as the latter user.
<br/>
It is quite simple to generate a shell script that can add automatically configuration lines so that all the PostgreSQL related commands will be executed:

<br/>
<br/>
```shell
# for cmd in /usr/local/bin/pg*; do
    echo "permit persist setenv { PGDATA=\$PGDATA } luca as postgres $cmd" >> //usr/local/etc/doas.conf
  done
```
<br/>
<br/>

and the above is going to produce something really verbose as:

<br/>
<br/>
```shell
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_archivecleanup
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_basebackup
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_checksums
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_config
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_controldata
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_ctl
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_dump
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_dumpall
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_isready
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_receivewal
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_recvlogical
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_repack
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_resetwal
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_restore
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_rewind
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_standby
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_test_fsync
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_test_timing
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_upgrade
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pg_waldump
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgbackrest
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgbadger
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgbench
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgbench_helper.sh
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgxn
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgxn-3.7
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgxnclient
permit persist setenv { PGDATA=/postgres/12/data } luca as postgres /usr/local/bin/pgxnclient-3.7
```
<br/>
<br/>

Of course, you can tune such *generator* as much as you like.


## Using commands against a single cluster (don't try this at home!)

In the previous examples, `doas` has been configured to allow only PostgreSQL related commands with a *default* `PGDATA` environment variable, but the user is still able to execute a command using a different directory:


<br/>
<br/>
```shell
% doas -u postgres pg_ctl -D /postgres/13/ start
waiting for server to start....
 done
server started
```
<br/>
<br/>

As you can configure `sudo`, you can also tune `doas` to accept only a specific data directory as option to the commands. This is, however, quite complex and prone to errors: you have to specify the environment and all available arguments, such as:

<br/>
<br/>
```shell
permit nopass setenv { PGDATA=$PGDATA } luca as postgres cmd  /usr/local/bin/pg_ctl args start
permit nopass setenv { PGDATA=$PGDATA } luca as postgres cmd  /usr/local/bin/pg_ctl args stop
permit nopass setenv { PGDATA=$PGDATA } luca as postgres cmd  /usr/local/bin/pg_ctl args restart
```
<br/>
<br/>

The situation becomes:

<br/>
<br/>
```shell
 % doas -u postgres /usr/local/bin/pg_ctl start
waiting for server to start....
...
 done
server started


% doas -u postgres /usr/local/bin/pg_ctl reload
doas: Operation not permitted
```
<br/>
<br/>

Please be aware that this is not a good solution however, because while updating the `doas.conf` file the file could result shorter and the rules could be executed in a way you don't figure.
<br/>
A better approach is, of course, allow the user to become `postgres` and have the latter able to do only her own tasks.


## Being able to run as user `postgres`

This is much simpler you may think and it resolves into the single rule:

<br/>
<br/>
```shell
permit persist setenv { PGDATA=$PGDATA } luca as postgres
```
<br/>
<br/>

Without specifying any command with the special keywor `cmd`, the user `luca` will be able to run any command as `postgres`, and such user will be able to execute every PostgreSQL related command.


# Conclusions

`doas(1)` is a nice piece of code that allows for a more readable and less tunable configuration than `sudo`, and this can be exploited to allow users for executing operations against PostgreSQL, among other programs.

