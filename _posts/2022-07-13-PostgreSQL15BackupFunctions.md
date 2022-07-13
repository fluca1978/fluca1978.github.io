---
layout: post
title:  "PostgreSQL 15: changes in the low level backup functions"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
The upcoming new release of PostgreSQL does some changes to low level backup functions.

# PostgreSQL 15: changes in the low level backup functions

The upcoming PostgreSQL 15 release implements a few changes into the /low level/ backup functions.
<br/>
Nowdays I suspect nobody, except backup solution developers, know or use such functions, but I clearly remember when we developed our own scripts to do a continuos backup using functions like `pg_start_backup()` and `pg_stop_backup()`.
<br/>
You should use other backup solutions today, like the great [pgBackRest](https://pgbackrest.org/){:target="_blank"}.
<br/>
In any case, what are the changes?
<br/>
As you can read from [the release notes](https://www.postgresql.org/docs/15/release-15.html{:target="_blank"} there two mainly:
1. *functions have been renamed* to a more consistent naming scheme.
2. a few functions and modes have been removed.

The [functions are now named as `pg_backup_`](https://www.postgresql.org/docs/15/continuous-archiving.html#BACKUP-LOWLEVEL-BASE-BACKUP){:target="_blank"}, so **`pg_start_backup()` becomes `pg_backup_start()`**, and similarly, **`pg_stop_backup()` becomes `pg_backup_stop()`**. Quite frankly I like this decision, it makes the naming simpler to search for and to remember.
<br/>
Moreover, there is no more the presence of deprecated (since version 9.6, if I remember correctly), the *exclusive backup mode*. This was the only way to perform a low level backup back in the days, but since a lot it has been deprecated. One of the problems with exclusive backups is that the system will create a label file that prevents the primary to restart after a crash, and in turn this led people to delete the label file also on standby servers.
Now this is no more a problem, and the `pg_backup_start()` and `pg_backup_stop()` functions do not handle anymore the exclusive backup parameter.
<br/>
As a consequence of this choice, the functions `pg_is_in_backup()` and `pg_backup_start_time()` have been removed because *they were focused only on exclusive backups*, that do not exist anymore.
