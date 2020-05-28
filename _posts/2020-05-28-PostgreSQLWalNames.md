---
layout: post
title:  "WAL, LSN and File Names"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Understanding the relationship between LSN and WAL file names.

# WAL, LSN and File Names

PostgreSQL stores changes that is going to apply to data into the **Write Ahead Logs (WALs)**, that usually are `16 MB` each in size, even if you can configure your cluster (starting from version 11) to different sizes.
<br/>
PostgreSQL knows at which part of the `16 MB` file (named *segment*) it is by an offset that is tied to the **Log Sequence Number (LSN)**. Let's see those in action.
<br/>
First of all, let's get some information about the current status:


```sql
testdb=> SELECT pg_current_wal_lsn(),
          pg_walfile_name( pg_current_wal_lsn() );;
-[ RECORD 1 ]------|-------------------------
pg_current_wal_lsn | C/CE7BAD70
pg_walfile_name    | 000000010000000C000000CE
```

The server is currently using the WAL file named `000000010000000C000000CE`.
It is possible to see the relationship between the LSN, currently `C/CE7BAD70` and the WAL file name as follows.
The LSN is made up by three pieces: `X/YYZZZZZZ` where:
- `X` represents the middle part of the WAL file name, one or two symbols;
- `YY` represents the final part of the WAL file name;
- `ZZZZZZ` are six symbols that represents the offset within the file name.
<br/>
Therefore, given the LSN `C/CE7BAD70` we can assume that the middle part of the WAL file name will be `C` and the last part will be `CE`, both zero padded to 8 symbols, so respectively `0000000C` and `000000CE`. Concatenated togehter, they provide us with a file name that ends with `0000000C000000CE`. The initial part of the filename is still missing, and that is the timeline the server is running on, in this case `1`, zero padded as the other parts, so `00000001` that provides us the final name `000000010000000C000000CE`.
<br/>
To summarize, the following is the correspondance between the single parts:

```shell
LSN  ->              C  /     CE      7BAD70
WAL  -> 00000001 0000000C 000000CE
```

<br/>
<br/>
> Please consider that the above example is just to show you the concept, but it is better to
> use the function pg_walfile_name() to get the exact WAL file name from an LSN since WAL switch
> may lead to incorrect result from the LSN "manual decoding".
<br/>
<br/>
The final part of the LSN is the offset within the WAL file, and it does suffice to convert it to `int` to get an idea:

```sql
testdb=> SELECT ( x'7BAD70' )::int AS offset;
-[ RECORD 1 ]---
offset | 8105328
```

You can get the same information with the special function `pg_walfile_name_offset()`, to which you can pass the LSN, and get the current filename and the offset in a single run:

```sql
testdb=> SELECT ( x'7BAD70' )::int AS offset_computed, pg_walfile_name_offset( 'C/CE7BAD70' );
-[ RECORD 1 ]----------|-----------------------------------
offset_computed        | 8105328
pg_walfile_name_offset | (000000010000000C000000CE,8105328)
```

<br/>
To summarize, given a specific LSN the database is (and must be) clearly aware of the WAL file segment the LSN refers to and to the exact offset, within such file, where the data can be found.
