---
layout: post
title:  "pgagroal new commands: 'ping' and ìstatus details'"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Another little improvement to the interface for `pgagroal`

# pgagroal new commands: 'ping' and ìstatus details'

When I [committed the major command refactoring](https://github.com/agroal/pgagroal/commit/ade40240317bad155dbf1e40866c96257b688b90){:target="_blank"} in `pgagroal-cli`, I introduced also a simple way to *deprecate a command*, so that the user running the old version of a command is warned about switching to the new interface.

This lead me to think I can not only refactor `pgagroal-cli` commands in a more coherent way, grouping similar commands together, but I can also change existing commands by means of deprecating them.

That is what I did [in this commitLink Text](https://github.com/agroal/pgagroal/commit/5f15164b8eff7063445ac454baefa4b4242c962f){:target="_blank"}  where I replaced the `is-alive` command with `ping` and `details` with `status details`.

## The `ping` command

I have to confess: the name has been inspired by the MySQL Admin tool, that has a similar command.

The idea of `ping` is to test if the connection pooler is alive, and it replaces the old command `is-alive`:


<br/>
<br/>
```shell
$ pgagroal-cli ping --verbose
pgagroal-cli: Success (0)


$ pgagroal-cli is-alive --verbose
pgagroal-cli: command <is-alive> has been deprecated by <ping> since version 1.6
pgagroal-cli: Success (0)

```
<br/>
<br/>


Please note that, as documented, the `ping` command does not print anything if the pooler is running.


## The `status details` command

The `status` command prints a summary information about the pooler, while the `details` command prints the same summary and a more verbose and detailed information about every connection.

Why not group these two commands? This is the aim of having `status details`:
- `status` will work as before;
- `status` enhanced with `details` will provide more verbose output.

<br/>
<br/>
```shell
$ pgagroal-cli status
Status:              Running
Active connections:  0
Total connections:   0
Max connections:     15


$ pgagroal-cli status details
Status:              Running
Active connections:  0
Total connections:   0
Max connections:     15
---------------------
Server:              venkman
Host:                venkman
Port:                5432
State:               Not init
---------------------
---------------------
Server:              a
Host:                spengler
Port:                5432
State:               Not init
---------------------
---------------------
Server:              b
Host:                spengler
Port:                6432
State:               Not init
---------------------
---------------------
Database:            testdb
Username:            luca
Active connections:  0
Max connections:     2
Initial connections: 1
Min connections:     1
---------------------
---------------------
Database:            all
Username:            luca
Active connections:  0
Max connections:     10
Initial connections: 2
Min connections:     1
---------------------
---------------------
Database:            pgbench
Username:            pgbench
Active connections:  0
Max connections:     2
Initial connections: 1
Min connections:     1
---------------------
Connection    0:     Not init
Connection    1:     Not init
Connection    2:     Not init
Connection    3:     Not init
Connection    4:     Not init
Connection    5:     Not init
Connection    6:     Not init
Connection    7:     Not init
Connection    8:     Not init
Connection    9:     Not init
Connection   10:     Not init
Connection   11:     Not init
Connection   12:     Not init
Connection   13:     Not init
Connection   14:     Not init
```
<br/>
<br/>

Therefore, now the total number of main commands to `pgagroal-cli` has shrinked, since a few of them have been grouped.


# Conclusions

While it may sound very trivial, having a coherent and easy to understand command line interface is a key value in make the project been approached by mere mortals. That's why I strongly believe the refactoring of the commands in `pgagroal-cli` is going to play a very important role in the connection pooler adoption.
