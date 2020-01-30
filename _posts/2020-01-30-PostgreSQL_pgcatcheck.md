---
layout: post
title:  "Checking catalogues for corruption with pg_catcheck"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
I just discovered a new utility for checking the health of a cluster.

# Checking catalogues for corruption with pg_catcheck

Today I discovered a nice tool from [EnterpriseDB](https://www.enterprisedb.com/){:target="_blank"} named [pg_catcheck](https://github.com/EnterpriseDB/pg_catcheck){:target="_blank"} that aims at checking the health of the PostgreSQL catalogs.
<br/>
As you know, if the catalogs are damaged, the database can quickly get confused and not allow you to use as you wish. Luckily, this is something does not happen very often, or rather I should say I think I've seen this happening only once during my career (and I don't remember the cause).
<br/>
While I'm not sure I would be able to fix any problem in the catalogues by myself, having a tool that helps me understanding if everything is fine is a relief!


## Installing pg_catcheck on FreeBSD

You need to get it from the [project repository](https://github.com/EnterpriseDB/pg_catcheck){:target="_blank"}. There is at the moment one official release, but let's use the `HEAD` (after all, *releases are for feeble people!*).

```shell
% git clone https://github.com/EnterpriseDB/pg_catcheck.git
% cd pg_catcheck
% gmake
...
% sudo gmake install                                       
...
```

If everything works fine, you will end up with a program named `pg_catcheck` under `/usr/local/bin`.

## Using pg_catcheck

As you can imagine, **you need a database administrator** to perform the check. The application supports pretty much the same options than `psql` to connect, and there's an extra option `--postgresql` to indicate you are running against a vanilla PostgreSQL (on the other hand, with `--enterprisedb` the program will assume you are running an EnterpriseDB instance).

```shell
% pg_catcheck --postgresql -U postgres template1
progress: done (0 inconsistencies, 0 warnings, 0 errors)
```

That's it, if you see `0 inconsinstencies` your database is fine.
<br/>

You can see what the program checks with the `--verbose` option:

```shell
% pg_catcheck --postgresql -U postgres  -v template1         
verbose: detected server version 120001
verbose: assuming PostgreSQL server
verbose: preloading table pg_authid because it is required in order to check pg_namespace
verbose: loading table pg_namespace
verbose: checking table pg_namespace (6 rows)
verbose: loading table pg_collation
verbose: checking table pg_collation (1110 rows)
verbose: loading table pg_tablespace
verbose: checking table pg_tablespace (2 rows)
verbose: loading table pg_language
verbose: checking table pg_language (4 rows)
verbose: loading table pg_database
verbose: checking table pg_database (4 rows)
verbose: loading table pg_largeobject_metadata
verbose: checking table pg_largeobject_metadata (0 rows)
verbose: loading table pg_publication
verbose: checking table pg_publication (0 rows)
verbose: loading table pg_subscription
verbose: checking table pg_subscription (0 rows)
verbose: loading table pg_default_acl
verbose: checking table pg_default_acl (0 rows)
verbose: loading table pg_largeobject
verbose: checking table pg_largeobject (0 rows)
verbose: loading table pg_db_role_setting
verbose: checking table pg_db_role_setting (0 rows)
verbose: loading table pg_auth_members
verbose: checking table pg_auth_members (8 rows)
verbose: preloading table pg_class because it is required in order to check pg_type
verbose: loading table pg_type
verbose: checking table pg_type (406 rows)
verbose: loading table pg_proc
verbose: checking table pg_proc (2960 rows)
verbose: loading table pg_operator
verbose: checking table pg_operator (770 rows)
verbose: loading table pg_ts_parser
verbose: checking table pg_ts_parser (1 rows)
verbose: loading table pg_ts_config
verbose: checking table pg_ts_config (22 rows)
verbose: loading table pg_ts_template
verbose: checking table pg_ts_template (5 rows)
verbose: loading table pg_ts_dict
verbose: checking table pg_ts_dict (22 rows)
verbose: loading table pg_foreign_data_wrapper
verbose: checking table pg_foreign_data_wrapper (0 rows)
verbose: loading table pg_foreign_server
verbose: checking table pg_foreign_server (0 rows)
verbose: loading table pg_cast
verbose: checking table pg_cast (216 rows)
verbose: loading table pg_conversion
verbose: checking table pg_conversion (132 rows)
verbose: loading table pg_extension
verbose: checking table pg_extension (1 rows)
verbose: loading table pg_enum
verbose: checking table pg_enum (0 rows)
verbose: loading table pg_user_mapping
verbose: checking table pg_user_mapping (0 rows)
verbose: loading table pg_event_trigger
verbose: checking table pg_event_trigger (0 rows)
verbose: loading table pg_rewrite
verbose: checking table pg_rewrite (126 rows)
verbose: loading table pg_attrdef
verbose: checking table pg_attrdef (0 rows)
verbose: loading table pg_policy
verbose: checking table pg_policy (0 rows)
verbose: loading table pg_publication_rel
verbose: checking table pg_publication_rel (0 rows)
verbose: loading table pg_statistic_ext
verbose: checking table pg_statistic_ext (0 rows)
verbose: loading table pg_transform
verbose: checking table pg_transform (0 rows)
verbose: loading table pg_attribute
verbose: checking table pg_attribute (2913 rows)
verbose: loading table pg_foreign_table
verbose: checking table pg_foreign_table (0 rows)
verbose: loading table pg_inherits
verbose: checking table pg_inherits (0 rows)
verbose: loading table pg_aggregate
verbose: checking table pg_aggregate (136 rows)
verbose: loading table pg_ts_config_map
verbose: checking table pg_ts_config_map (418 rows)
verbose: loading table pg_statistic
verbose: checking table pg_statistic (422 rows)
verbose: loading table pg_init_privs
verbose: checking table pg_init_privs (171 rows)
verbose: loading table pg_sequence
verbose: checking table pg_sequence (0 rows)
verbose: loading table pg_subscription_rel
verbose: checking table pg_subscription_rel (0 rows)
verbose: preloading table pg_am because it is required in order to check pg_opfamily
verbose: loading table pg_opfamily
verbose: checking table pg_opfamily (107 rows)
verbose: checking table pg_class (395 rows)
verbose: loading table pg_opclass
verbose: checking table pg_opclass (128 rows)
verbose: loading table pg_amop
verbose: checking table pg_amop (715 rows)
verbose: loading table pg_amproc
verbose: checking table pg_amproc (447 rows)
verbose: loading table pg_index
verbose: checking table pg_index (159 rows)
verbose: loading table pg_constraint
verbose: checking table pg_constraint (2 rows)
verbose: loading table pg_trigger
verbose: checking table pg_trigger (0 rows)
verbose: loading table pg_range
verbose: checking table pg_range (6 rows)
verbose: loading table pg_depend
verbose: checking table pg_depend (7601 rows)
verbose: loading table pg_shdepend
verbose: checking table pg_shdepend (16 rows)
verbose: loading table pg_description
verbose: checking table pg_description (4744 rows)
verbose: loading table pg_shdescription
verbose: checking table pg_shdescription (3 rows)
verbose: loading table pg_seclabel
verbose: checking table pg_seclabel (0 rows)
verbose: loading table pg_shseclabel
verbose: checking table pg_shseclabel (0 rows)
verbose: loading table pg_partitioned_table
verbose: checking table pg_partitioned_table (0 rows)
progress: done (0 inconsistencies, 0 warnings, 0 errors)
```

Thanks [EnterpriseDB](https://www.enterprisedb.com/){:target="_blank"} named [pg_catcheck](https://github.com/EnterpriseDB/pg_catcheck){:target="_blank"} for making this tool open source!
