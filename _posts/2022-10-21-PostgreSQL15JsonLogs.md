---
layout: post
title:  "PostgreSQL 15: logging in JSON"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PostgreSQL 15 has now the capability to output logs in JSON format!

# PostgreSQL 15: logging in JSON

The [freshly released PostgreSQL 15](https://www.postgresql.org/docs/15/release-15.html){:target="_blank"} introduces a lot of new features and improvements, but one, according to me, is going to change the way our favourite database is monitored: [the capability to log daemon status in JSON](https://www.postgresql.org/docs/15/runtime-config-logging.html#GUC-LOG-DESTINATION){:target="_blank"}.

<br/>
<br/>
Essentially, the `log_destination` configuration parameter now has another enumerated value: **`jsonlog`**. When this value is added to `log_destination`, PostgreSQL will start to emit JSON structured logs.
Here it is a simple configuration example:

<br/>
<br/>
```shell
% grep log_destination /postgres/15/data/postgresql.conf
log_destination = 'stderr,jsonlog'
```
<br/>
<br/>

and this is how the `log` directory appears right after the configuration has been reloaded:

<br/>
<br/>
```shell
% sudo -u postgres ls /postgres/15/data/log -1
postgresql-Fri.json
postgresql-Fri.log
```
<br/>
<br/>

Clearly the logs will contain the same values, but in different formats.

<br/>
<br/>

There is more: when there's more than one value set in `log_destination`, PostgreSQL will store a file named **`current_logfiles`**, where each line will represent the format and the current logfile where PostgreSQL has to store the data:

<br/>
<br/>
```shell
% sudo -u postgres cat /postgres/15/data/current_logfiles
stderr log/postgresql-Fri.log
jsonlog log/postgresql-Fri.json

```
<br/>
<br/>

In this way, not only PostgreSQL, but even the sysadmin can keep track of where the system is going to log right now, and this is useful especially when there's a log rotation in place.

<br/>
<br/>
On the SQL side, the function `pg_current_logfile()` can optionally accept the log format (the same specified in `log_destination`) and provide the current log file depending on the choosen format:


<br/>
<br/>
```sql
testdb=# select pg_current_logfile();
   pg_current_logfile
------------------------
 log/postgresql-Fri.log

testdb=# select pg_current_logfile( 'jsonlog' );
   pg_current_logfile
-------------------------
 log/postgresql-Fri.json

testdb=# select pg_current_logfile( 'stderr' );
   pg_current_logfile
------------------------
 log/postgresql-Fri.log

```
<br/>
<br/>

I suspect we will see more and more *log crunching* applications to switch over the new JSON log format!
