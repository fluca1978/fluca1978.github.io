---
layout: post
title:  "PostgreSQL to Microsoft SQL Server Using TDS Foreign Data Wrapper"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
I needed to push data from a Microsoft SQL Server 2005 to our beloved database, so why don't use a FDW to the purpose? It has not been as simple as with other FDW, but works!

PostgreSQL to Microsoft SQL Server Using TDS Foreign Data Wrapper
---

At work I needed to push data out from a Microsoft SQL Server 2005 to a PostgreSQL 11 instance. *Foreign Data Wrappers* was my first thought! *Perl to the rescue* was my second, but since I had some time, I decided to investigate the first way first.

<br/>
The scenario was the following:
- CentOS 7 machine running PostgreSQL 11, it is not my preferred setup (I do prefer either FreeBSD or Ubuntu), but I have to deal with that;
- Microsoft SQL Server 2005 running on a Windows Server <something>, surely not something I like to work with and to which I had to connect via remote desktop (argh!).


# First Step: get TDS working

After a quick research on the web, I discovered that MSSQL talks the **Table Data Stream** (TDS for short), so I don't need to install an ODBC stack on my Linux box. And luckily, there are binaries for CentOS:

```shell
$ sudo yum install freetds
$ sudo yum install freetds-devel freetds-doc
```

`freetds` comes along with a `tsql` terminal command that is meant to be a diagnosing tool, so nothing as complete as a `psql` terminal. **You should really test your connectivity with `tsql` before proceeding further**, since it can save you hours of debugging when things do not work.
<br/>
Thanks to a pragmatic test with `tsql` I discovered that I needed to open port `1433` (default MSSQL port) on our enterprise firewall, as well as I had to create a database user on the MSSQL instance (not the right term, I know) and to grant read permissions to it. After that, I spent half an hour trying to understand how to send queries from `tsql` to MSSQL, because the former uses the special `go` keyword to send queries (something like `psql` `\g`):

```shell
$ tsql -H 192.168.6.53 -p 1433 -U 'fluca1978' -P 'xxxx'
1> select name from [mydb].[dbo].[people]
2> go
...
```

Once I got `tsql` connection working, was time to install the foreign data wrapper.

# Step 2: Install `tds_fdw`

The foreign data wrapper to connect to MSSQL is an FDW that exploits TDS: [tds_fdw](https://github.com/tds-fdw/tds_fdw/). Unluckily, binaries are for older PostgreSQL versions and, moreover, there is [a problem that prevents compilation against PostgreSQL 11](https://github.com/tds-fdw/tds_fdw/issues/192). Luckily there is already a patch, so it is recommended to compile the current HEAD:

```shell
$ git show
commit 3719a995b0ae3fc4c4b390dd8a2820d54b88e18a
Merge: 782883d 3909a44
Author: Geoff Montee <geoff.montee@gmail.com>
Date:   Thu Jan 3 15:21:14 2019 -0800

    Merge pull request #190 from l-we/patch-1

    Update options.c


$ make
...
$ sudo make install
...
/usr/bin/mkdir -p '/usr/pgsql-11/lib'
/usr/bin/mkdir -p '/usr/pgsql-11/share/extension'
/usr/bin/mkdir -p '/usr/pgsql-11/share/extension'
/usr/bin/mkdir -p '/usr/pgsql-11/doc/extension'
/usr/bin/install -c -m 755  tds_fdw.so '/usr/pgsql-11/lib/tds_fdw.so'
/usr/bin/install -c -m 644 .//tds_fdw.control '/usr/pgsql-11/share/extension/'
/usr/bin/install -c -m 644 .//sql/tds_fdw--2.0.0-alpha.2.sql  '/usr/pgsql-11/share/extension/'
/usr/bin/install -c -m 644 .//README.tds_fdw.md '/usr/pgsql-11/doc/extension/'
/usr/bin/mkdir -p '/usr/pgsql-11/lib/bitcode/tds_fdw'
/usr/bin/mkdir -p '/usr/pgsql-11/lib/bitcode'/tds_fdw/src/
/usr/bin/install -c -m 644 src/tds_fdw.bc '/usr/pgsql-11/lib/bitcode'/tds_fdw/src/
/usr/bin/install -c -m 644 src/options.bc '/usr/pgsql-11/lib/bitcode'/tds_fdw/src/
/usr/bin/install -c -m 644 src/deparse.bc '/usr/pgsql-11/lib/bitcode'/tds_fdw/src/
cd '/usr/pgsql-11/lib/bitcode' && /usr/lib64/llvm5.0/bin/llvm-lto -thinlto -thinlto-action=thinlink -o tds_fdw.index.bc tds_fdw/src/tds_fdw.bc tds_fdw/src/options.bc tds_fdw/src/deparse.bc
```

# Step 3: Use the FDW

With the foreign data wrapper in place, it is now time to create PostgreSQL objects. **I recommend using `notice` as a message level because it can provide valuable information about the connection to the foreign server!** In my case, it helped me understand that the *showplan* was something I needed to grant permission on to my MSSQL user, otherwise the conenction will fail with a generic "See the server log", which made me discover that Microsoft is not very good at logging!

```sql
testdb=# CREATE EXTENSION tds_fdw;
testdb=# CREATE SERVER
   server_mssql
   FOREIGN DATA WRAPPER tds_fdw
   OPTIONS( servername '192.168.6.53',
   database 'mydb',
   msg_handler 'notice',  -- useful!
   tds_version '7.2'      -- MSSQL 2005
);

testdb=# CREATE USER MAPPING FOR luca
	SERVER server_mssql
	OPTIONS (
       username 'fluca1978',
       password 'xxxx'
);

testdb=# CREATE FOREIGN TABLE mssql_people
(
   ...
)
SERVER server_mssql
OPTIONS (
  schema_name 'dbo',
  table_name 'people',
  --row_estimate_method 'showplan_all' -- requires extra privileges on MSSQL side!
);
```

# Step 4: Use

If everything goes ok, it is possible to query `mssql_people` and get some `notice` messages about retrieving data:

```sql
testdb=# select * from people limit 5
NOTIFICA:  DB-Library notice: Msg #: 5701, Msg state: 2, Msg: Changed database context to 'mydb'., Server: 192.168.6.53, Process: , Line: 1, Level: 0
NOTIFICA:  DB-Library notice: Msg #: 5703, Msg state: 1, Msg: Changed language setting to us_english., Server: 192.168.6.53, Process: , Line: 1, Level: 0
NOTIFICA:  tds_fdw: Query executed correctly
NOTIFICA:  tds_fdw: Getting results
NOTIFICA:  DB-Library notice: Msg #: 5701, Msg state: 2, Msg: Changed database context to 'mydb'., Server: 192.168.6.53, Process: , Line: 1, Level: 0
NOTIFICA:  DB-Library notice: Msg #: 5703, Msg state: 1, Msg: Changed language setting to us_english., Server: 192.168.6.53, Process: , Line: 1, Level: 0

...
(5 rows)

Time: 192231,473 ms (03:12,231)
```

Times are really bad, but this is due to MSSQL database design I'm working with, so I don't blame nor MSSQL itself, nor `tds_fdw`: the table is a few gigabytes in size. I really need to investigate some optimizations.
