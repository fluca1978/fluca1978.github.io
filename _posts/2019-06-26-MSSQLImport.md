---
layout: post
title:  "Importing data from MSSQL is faster than I thought"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A few months ago I set up a Foreign Data Wrapper against a Microsoft SQL Server to import historical data.
I'm quite impressive about how quick the bulk import is.

# Importing data from MSSQL is faster than I thought

It has not been an exciting job, but having PostgreSQL to pull data out of Microsoft SQL Server is a joy!
The architecture is quite dumb:
- PostgreSQL connects via *Foreign Data Wrapper* to the MSSQL machine;
- a function is executed to extract records, crunch them and store modified into the PostgreSQL engine;
- every 10 minutes, iterate.

So far, it is importing `3,5k` tuples every ten minutes, that is around `500000` tuples per day. So far it is running in less than a second, and I'm able to monitor that thanks to a *status* table where I store the `clock_timestamp()` values for monitoring:

```sql
testdb=> SELECT record_count, 
             ts_end - ts_begin AS elapsed_time
         FROM pull_stat ORDER BY pk DESC LIMIT 10;
 record_count |  elapsed_time   
--------------|-----------------
         3659 | 00:00:00.440777
         3656 | 00:00:00.363089
         3694 | 00:00:00.385919
         3713 | 00:00:00.460304
         3695 | 00:00:02.209158
         3678 | 00:00:00.393815
         3699 | 00:00:00.404685
         3693 | 00:00:00.403348
         3704 | 00:00:00.358293
         3683 | 00:00:00.355856
         
testdb=> select version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 11.1 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-28), 64-bit
```

While this is surely not a benchmark, I'm quite impressed about the speed of pulling data from a foreign server.
It's interesting to note that the `tds` version is `7.2` with `notice` messages enabled (that I suspect lead to a little time expense).

```sql
testdb=# \des+
List of foreign servers

Name                 | server_mssql
Owner                | postgres
Foreign-data wrapper | tds_fdw
Access privileges    | 
Type                 | 
Version              | 
FDW options          | (servername '10.0.0.1', database 'AXES', msg_handler 'notice', tds_version '7.2')
Description          | 
```
