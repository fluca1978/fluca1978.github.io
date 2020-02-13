---
layout: post
title:  "Take advantage of pg_settings when dealing with your configuration"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
The right way to get the current PostgreSQL configuration is by means of pg_settings.

# Take advantage of pg_settings when dealing with your configuration

I often see messages on PostgreSQL related mailing list where the configuration is assumed by a *Unix-style* approach. For example, imagine you have been asked to provide your *autovacuum* configuration in order to see if there's something wrong with it; one approach I often is the copy and paste of the following:

```
% sudo -u postgres grep autovacuum /postgres/12/postgresql.conf
#autovacuum_work_mem = -1               # min 1MB, or -1 to use maintenance_work_mem
#autovacuum = on                        # Enable autovacuum subprocess?  'on'
#log_autovacuum_min_duration = -1       # -1 disables, 0 logs all actions and
autovacuum_max_workers = 7             # max number of autovacuum subprocesses
autovacuum_naptime = 2min              # time between autovacuum runs
autovacuum_vacuum_threshold = 500       # min number of row updates before
autovacuum_analyze_threshold = 700      # min number of row updates before
#autovacuum_vacuum_scale_factor = 0.2   # fraction of table size before vacuum
#autovacuum_analyze_scale_factor = 0.1  # fraction of table size before analyze
#autovacuum_freeze_max_age = 200000000  # maximum XID age before forced vacuum
#autovacuum_multixact_freeze_max_age = 400000000        # maximum multixact age
#autovacuum_vacuum_cost_delay = 2ms     # default vacuum cost delay for
                                        # autovacuum, in milliseconds;
#autovacuum_vacuum_cost_limit = -1      # default vacuum cost limit for
                                        # autovacuum, -1 means use
```

While this *could be a correct approach* and makes it simply to provide a *full set* of configuration values, it has few drawbacks:
- it produces verbose output (e.g., there are comments on the right of each line);
- *it could not be the whole story about the configuration*, for example because something is in  `postgresql.conf.auto`;
- **it does include commented out lines**;
- **it could be not the configuration your cluster is running on**.

Let's examine all the drawbacks, one at a time.

## Verbose Output

This is much annoyance than a real problem, but please consider that people on the other part of the world could have a screen resolution, line wrapping, or setting that makes it difficult to read verbose lines.

## Could not be the whole truth about configuration

I often place my own PostgreSQL configuration into `include_if_exists` files, so that I leave the `postgresql.conf` file unchanged. Let's call it a kind of FreeBSD configuration style!
<br/>
This means that, in order to use a Unix approach to find a particular setting, I have to include in the search every single configuration file in every single location. This can be as simple as doing `grep autovacuum *.conf*` or much more complicated depending on your directory structure.
<br/>
In any case, I could have omitted one single file, and that's bad both for me and whoever is trying to help me.
<br/>
Moreover, since `ALTER SYSTEM` is gaining more and more power, setting could also live into `postgresql.conf.auto` and people should begin used to check also such file.

## It does include commented-out lines

Come on, who cares about that? After all commented-out lines means the value is at its default value.
<br/>
And that could be the problem: do you remember all the default values for every single PostgreSQL version?
<br/>
Therefore, don't trust the default value, get the exact value!

## It could not be the configuration you cluster is using

What if you modified the configuration file but edited the wrong context with regard to the action you did? May be you edited a `postmaster` context parameter and issued only a *simple* `SIGHUP`? What if you forgot to notify the cluster at all?
<br/>
What if another administrator changed the parameters without telling you and scheduling a *cluster notification* at night? Yes, that really happened to me...
<br/>
Again, get the real configuration!

# How to get the real configuration

I'm glad you asked: `pg_settings` is there for you.
<br/>
It is a matter of a single query, for example:

```sql
forumdb=> SELECT name, setting, pending_restart 
          FROM pg_settings 
          WHERE name like 'autovacuum%' 
          ORDER BY 1;
                name                 |  setting  | pending_restart 
-------------------------------------|-----------|-----------------
 autovacuum                          | on        | f
 autovacuum_analyze_scale_factor     | 0.1       | f
 autovacuum_analyze_threshold        | 50        | f
 autovacuum_freeze_max_age           | 200000000 | f
 autovacuum_max_workers              | 3         | f
 autovacuum_multixact_freeze_max_age | 400000000 | f
 autovacuum_naptime                  | 60        | f
 autovacuum_vacuum_cost_delay        | 2         | f
 autovacuum_vacuum_cost_limit        | -1        | f
 autovacuum_vacuum_scale_factor      | 0.2       | f
 autovacuum_vacuum_threshold         | 50        | f
 autovacuum_work_mem                 | -1        | f
(12 rows)

```
You can elaborate the query as you like, but the point is that you get exact values. In this particular example, as you can see, some values differs from what you get out of the configuration file. For example, `autovacuum_max_worker` has been set to `7` in the configuration file, but the database applies a value of `3`.
<br/>
Now you can inspect this problem too, and see if it has been caused from a cluster that has not been notified about configuration changes or an included file that overwrites your settings.


# Conclusions

The configuration file is always only an hint about what your cluster is configured for, not the real thruth.
When inspecting a configuration problem, the starting point *to report even to others* is the output of `pg_settings`.
