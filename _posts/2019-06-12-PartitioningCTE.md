---
layout: post
title:  "A recursive CTE to get information about partitions"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
I was wondering about writing a function that provides a quick status about partitioning. But wait, PostgreSQL has recursive CTEs!

# A recursive CTE to get information about partitions
I'm used to partitioning, it allows me to quickly and precisely split data across different tables.
PostgreSQL 10 introduced the native partitioning, and since that I'm using native partitioning over inheritance whenever it is possible.
<br/>
But how to get a quick overview of the partition status? I mean, knowing which partition is growing the more?
<br/>
In the beginning I was thinking to write a function to do that task, quickly finding myself iterating recursively over `pg_inherits`, the table that *links* partitions to their parents. But the keyword here is *recursively*: PostgreSQL provides *recursive Common Table Expression*, and a quick search revelead I was right: it is possible to do it with a single CTE. Taking inspiration from [this mailing list message](https://www.postgresql.org/message-id/otalb9%245ma%241%40blaine.gmane.org), here it is a simple CTE to get a partition status (you can find it on my [GitHub repository](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/partitioning_report.sql)):

```sql
WITH RECURSIVE inheritance_tree AS (
     SELECT   c.oid AS table_oid
            , c.relname  AS table_name
            , NULL::text AS table_parent_name
            , c.relispartition AS is_partition
     FROM pg_class c
     JOIN pg_namespace n ON n.oid = c.relnamespace
     WHERE c.relkind = 'p'
     AND   c.relispartition = false

     UNION ALL

     SELECT inh.inhrelid AS table_oid
          , c.relname AS table_name
          , cc.relname AS table_parent_name
          , c.relispartition AS is_partition
     FROM inheritance_tree it
     JOIN pg_inherits inh ON inh.inhparent = it.table_oid
     JOIN pg_class c ON inh.inhrelid = c.oid
     JOIN pg_class cc ON it.table_oid = cc.oid

)
SELECT
          it.table_name
        , c.reltuples
        , c.relpages
        , CASE p.partstrat
               WHEN 'l' THEN 'BY LIST'
               WHEN 'r' THEN 'BY RANGE'
               ELSE 'not partitioned'
          END AS partitionin_type
        , it.table_parent_name
        , pg_get_expr( c.relpartbound, c.oid, true ) AS partitioning_values
        , pg_get_expr( p.partexprs, c.oid, true )    AS sub_partitioning_values
FROM inheritance_tree it
JOIN pg_class c ON c.oid = it.table_oid
LEFT JOIN pg_partitioned_table p ON p.partrelid = it.table_oid
ORDER BY 1,2;
```

The bootstrap term in the CTE selects all the tables that are not partition, that is the *roots* of a partitioning scheme. The recursive term simply joins `pg_inherits` in order to extract the children information.
The query attached to the CTE extracts information like the number of tuples and pages (that's what I need), and a summary of the partitioning including second level partitioning. Thanks to `pg_get_expr` it is possible to get a human readable partitioning startegy.
<br/>
<br/>
The output results in something like the following:

```sql
...
-[ RECORD 5 ]-----------|----------------------------------
table_name              | y2018
reltuples               | 0
relpages                | 0
partitionin_type        | BY LIST
table_parent_name       | root
partitioning_values     | FOR VALUES IN ('2018')
sub_partitioning_values | date_part('month'::text, mis_ora)
...
-[ RECORD 15 ]----------|----------------------------------
table_name              | y2018m10
reltuples               | 1.48956e+07
relpages                | 139212
partitionin_type        | not partitioned
table_parent_name       | y2018
partitioning_values     | FOR VALUES IN ('10')
sub_partitioning_values | 
```

That states table `y2018` is child of table `root`, accepts values `'2018'` and is partitioned by list, and children are partitioned by month. On the other hand, `y2018m10` is not partitioned anymore and is child of `y2018'`.
<br/>
That's a quick glance at the partitioning status in the cluster! Of course, it is possible to improve on this to get more information or restrict it depending on your needs.
