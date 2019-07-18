---
layout: post
title:  "Checking PostgreSQL Version in Scripts"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
`psql(1)` has a bar support for conditionals, that can be used to check PostgreSQL version and act accordingly in scripts.

# Checking PostgreSQL Version in Scripts
[`psql(1)` provides a little support to conditionals](https://www.postgresql.org/docs/11/app-psql.html) and this can be used in scripts to check, for instance, the PostgreSQL version.
<br/>
This is quite trivial, however I had to adjust an example script of mine to act properly depending on the PostgreSQL version.

## The problem
The problem I had was with declarative partitioning: since PostgreSQL 11, declarative partitioning supports a `DEFAULT` partition, that is *catch-all bucket* for tuples that don't have an explicit partition to go into.
In PostgreSQL 10 you need to manually create *catch-all* partition(s) by explicitly defining them.
<br/>
In my use case, I had a set of tables partitioned by a time range (the year, to be precise), but I don't want to set up a partition for each year before the starting point of *clean data*: all data after year 2015 is correct, somewhere there could be some dirty data with bogus years.
<br/>
Therefore, I needed a partition to catch all bogus data before year 2015, that is, a partition that ranges from the earth creation until 2015. In PostgreSQL 11 this, of course, requires you to define a `DEFAULT` partition and that's it! But how to create a different *default* partition on PostgreSQL 10 and 11?
<br/>
<br/>
 [I solved the problem with something like the following](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/partitioning/partitioning_example.declarative.sql):

```sql
\if :pg_version_10

\echo 'PostgreSQL version is 10'
\echo 'Emulate a DEFAULT partition'

CREATE TABLE digikam.images_old
       PARTITION OF digikam.images_root
       FOR VALUES FROM ( MINVALUE )
                TO ( '2015-01-01' );

\else

\echo 'PostgreSQL version is at least 11'
\echo 'Using DEFAULT partition'

CREATE TABLE digikam.images_old
       PARTITION OF digikam.images_root
       DEFAULT;

\endif
```

The idea is quite simple: if (`\if`) PostgreSQL is at version 10 emulate a default partition, otherwise (`\else`) PostgreSQL is at version 11 or greater and can use native `DEFAULT` partition. The partition table is named the same in the two cases so that the final user does not see any difference.
<br/>
<br/>
But what is that `:pg_version_10` stuff? That's a boolean `psql(1)` variable set up by another [utility](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/pgsql.check_postgresql_version.psql), included into my script:

```sql
SELECT
EXISTS ( SELECT setting
         FROM   pg_settings
         WHERE  name = 'server_version_num'
         AND    setting::int >= 120000
         AND    setting::int  < 130000
       )
       AS pg_version_12
, EXISTS ( SELECT setting
         FROM   pg_settings
         WHERE  name = 'server_version_num'
         AND    setting::int >= 110000
         AND    setting::int  < 120000
         )
         AS pg_version_11
-- and so on ...
, EXISTS ( SELECT setting
         FROM   pg_settings
         WHERE  name = 'server_version_num'
         AND    setting::int < 100000
         )
         AS pg_version_less_than_10
\gset
```

The script does a very dummy job: it queries the `server_version_num` setting and dynamically creates (`\gset`) variables that are true depending on the PostgreSQL instance version number.
<br/>
The only thing required is to import the script, for instance at the very top of your script, as for instance:

```sql
-- beginning of your script
\ir ../pgsql.check_postgresql_version.psql
```
<br/>
<br/>
*And that's all folks!*
<br/>
<br/>
What this allows me to do is, for instance, avoid to run a declarative partition script at all if that is not supported on the server side:

```sql
\if :pg_version_less_than_10
\echo 'PostgreSQL version less than 10, cannot run declarative partitioning!'
\echo 'Update yourself!'
\quit
\endif
```

Just placing the above snippet on top of my declarative partitioning script prevents me to running commands that will generate errors if the server is not at least at version 10.

## Summary
Thanks to `psql(1)` conditionals support it is possible to behave differently depending on the server version. 
<br/>
The advantage is that, clearly, you can build more robust scripts. 
<br/>
The drawback is that such script will require `psql(1)` and are therefore less portable.
