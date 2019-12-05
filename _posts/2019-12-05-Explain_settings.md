---
layout: post
title:  "PostgreSQL 12 EXPLAIN SETTINGS"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
PostgreSQL 12 has a very interesting feature to turn on when doing an execution plan analysis.

# PostgreSQL 12 EXPLAIN SETTINGS

PostgreSQL 12 has a new feature that can be turned on in the `EXPLAIN` output: `SETTINGS`. This option provides some information about **all and only those** parameters that can affect an execution plan **if and only if** they are not at the default setting.
<br/>
What does it mean in practice? Let's see an *old plain `EXPLAIN`*:

```sql
digikamdb=> EXPLAIN (FORMAT YAML)
            SELECT * FROM digikam_images 
            WHERE id IN ( SELECT id FROM digikam_images 
                          WHERE modificationdate = '2019-10-04' );  
```

the output is as follows:

```yaml
 - Plan:                                                  +
     Node Type: "Nested Loop"                             +
     Parallel Aware: false                                +
     Join Type: "Inner"                                   +
     Startup Cost: 0.29                                   +
     Total Cost: 1737.95                                  +
     Plan Rows: 17                                        +
     Plan Width: 87                                       +
     Inner Unique: true                                   +
     Plans:                                               +
       - Node Type: "Seq Scan"                            +
         Parent Relationship: "Outer"                     +
         Parallel Aware: false                            +
         Relation Name: "digikam_images"                  +
         Alias: "digikam_images_1"                        +
         Startup Cost: 0.00                               +
         Total Cost: 1596.72                              +
         Plan Rows: 17                                    +
         Plan Width: 8                                    +
         Filter: "(modificationdate = '2019-10-04'::date)"+
       - Node Type: "Index Scan"                          +
         Parent Relationship: "Inner"                     +
         Parallel Aware: false                            +
         Scan Direction: "Forward"                        +
         Index Name: "idx_id"                             +
         Relation Name: "digikam_images"                  +
         Alias: "digikam_images"                          +
         Startup Cost: 0.29                               +
         Total Cost: 8.31                                 +
         Plan Rows: 1                                     +
         Plan Width: 87                                   +
         Index Cond: "(id = digikam_images_1.id)"
```

The output is quite long, as well as the query is intentionally stupid just to generate some kind of loop.
Please note that I'm using `yaml` as an output format for better web impagination.
<br/>
<br/>
Let's see `SETTINGS` in action, so change the `EXPLAIN` as follows:

```sql
digikamdb=> EXPLAIN (FORMAT YAML, SETTINGS ON)
            SELECT * FROM digikam_images 
            WHERE id IN ( SELECT id FROM digikam_images 
                          WHERE modificationdate = '2019-10-04' );  
```

that produces *the very same output*!
<br/>
Why? Because nothing has changed, so nothing must be shown!
<br/>
Now, let's change a parameter or two:

```sql
digikamdb=> SET seq_page_cost TO 3;
digikamdb=> SET random_page_cost TO 1;
```

and see again the `EXPLAIN` in action:

```sql
digikamdb=> EXPLAIN (FORMAT YAML, SETTINGS ON)
            SELECT * FROM digikam_images 
            WHERE id IN ( SELECT id FROM digikam_images 
                          WHERE modificationdate = '2019-10-04' );  
...
 - Plan:                                                  +
     Node Type: "Nested Loop"                             +
     Parallel Aware: false                                +
     Join Type: "Inner"                                   +
     Startup Cost: 0.29                                   +
     Total Cost: 4353.95                                  +
     Plan Rows: 17                                        +
     Plan Width: 87                                       +
     Inner Unique: true                                   +
     Plans:                                               +
       - Node Type: "Seq Scan"                            +
         ...
       - Node Type: "Index Scan"                          +
         ...
   Settings:                                              +
     random_page_cost: "1"                                +
     seq_page_cost: "4"

```


As you can see, there is another section at the end of the output, titled `Settings`, that reminds us what parameters have changed and to which value they are currently.
<br/>
<br/>
In this way, it is possible to get an idea of why a plan is as it is, or at least we can remember that the system is running with different parameters.


## Are all parameters affected?

Reading the documentation [about `SETTINGS`](https://www.postgresql.org/docs/12/sql-explain.html){:target="_blank""_"} one could think that only those parameters that are part of an access method are going to be reported on the output of `EXPLAIN`:

```
SETTINGS

    Include information on configuration parameters. 
    Specifically, include options affecting query planning 
    with value different from the built-in default value. 
    This parameter defaults to FALSE.

```

However, even parameters that are not going to change the query plan are displayed. For example, in selection all the tuples, there is no need to know that the random page cost has changed, but it is displayed anyway:

```sql
digikamdb=> RESET ALL;
digikamdb=> SET seq_page_cost TO 2;
digikamdb=> SET random_page_cost TO 1;
digikamdb=> EXPLAIN (SETTINGS ON) SELECT * FROM digikam_images;
                              QUERY PLAN                              
----------------------------------------------------------------------
 Seq Scan on digikam_images  (cost=0.00..2364.58 rows=55258 width=87)
 Settings: random_page_cost = '1', seq_page_cost = '2'

```

## Which parameters?

There are different parameters, other than the trivial *costs*, that can be reported by `SETTINGS` section. An example is `work_mem`. Reading the commit [ea569d64ac7174d3fe657e3e682d11053ecf1866](https://github.com/postgres/postgres/commit/ea569d64ac7174d3fe657e3e682d11053ecf1866){:target="_blank"} reveals that all the options marked in the source code with `GUC_EXPLAIN` are candidates to be printed.
<br/>
So far, this resolves to the following long list, where I tried to mark as bold those that I usually configure (and I've seen touched by others):
- **`enable_seqscan`, `enable_indexscan` `enable_indexonlyscan`,  `enable_bitmapscan`**;
- **`temp_buffers`,  `work_mem`**;
- **`max_parallel_workers_per_gather`,  `max_parallel_workers`, `enable_gathermerge`**;
- **`effective_cache_size`**;
- **`min_parallel_table_scan_size`,  `min_parallel_index_scan_size`**;
- **`enable_parallel_append`, `enable_parallel_hash`, `enable_partition_pruning`**;
- **`enable_nestloop`, `enable_mergejoin`, `enable_hashjoin`**;
- `enable_tidscan`;
- `enable_sort`;
- `enable_hashagg`;
- `enable_material`;
- `enable_partitionwise_join`;
- `enable_partitionwise_aggregate`;
- `geqo`;
- `optimize_bounded_sort`;
- `parallel_leader_participation`;
- `jit`;
- `from_collapse_limit`;
- `join_collapse_limit`;
- `geqo_threshold`;
- `geqo_effort`;
- `geqo_pool_size`;
- `geqo_generations`;
- `effective_io_concurrency`;


## What about `auto_explain`?

The new `SETTINGS` affects also the `auto_explain` tuning and output, and in fact there is a new GUC named [`auto_explain.log_settings`](https://www.postgresql.org/docs/12/auto-explain.html) that provides the same functionality as above for the `auto_explain` module.

# Conclusions

The `EXPLAIN (SETTINGS ON)` new feature is something really cool in my opinion that pretty much every DBA should turn on when inspecting query execution plans.
