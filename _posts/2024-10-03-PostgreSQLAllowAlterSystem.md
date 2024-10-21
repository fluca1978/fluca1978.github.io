---
layout: post
title:  "PostgreSQL 17 allow_alter_system tunable"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PostgreSQL 17 includes a new (among others) tunable to control the `ALTER SYSTEM` command.

# PostgreSQL 17 allow_alter_system tunable

Among the new excellent features of PostgreSQL 17, one captured my attention: [the capability to disable the ALTER SYSTEM command](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-ALLOW-ALTER-SYSTEM){:target="_blank"} via the tunable `[allow_alter_system](https://www.postgresql.org/docs/current/runtime-config-compatible.html#GUC-ALLOW-ALTER-SYSTEM){:target="_blank"}`.

The `allow_alter_system` is a boolean setting that is turned `on` by default, meaning that it is always possible to execute `ALTER SYSTEM` on the enrironment (as in previous versions). When turned `off`, the system will report an error, refusing to execute the command:

<br/>
<br/>
```sql
postgres=# alter system set work_mem to '512MB';
ERROR:  ALTER SYSTEM is not allowed in this environment

postgres=# show allow_alter_system ;
 allow_alter_system
--------------------
 off
(1 row)

```
<br/>
<br/>

The idea, as explained in the documentation, is to prevent mistakes when PostgreSQL is managed externally, or with an external tool, so that it is not possible to accidentally overwrite a configuration managed outside the database itself (i.e., via traditional files).

The annotation for the tunable explains it:

<br/>
<br/>
```sql
postgres=# select name, context, category, short_desc, extra_desc from pg_settings where name = 'allow_alter_system';
-[ RECORD 1 ]--------------------------------------------------------------------------------------------------------------
name       | allow_alter_system
context    | sighup
category   | Version and Platform Compatibility / Other Platforms and Clients
short_desc | Allows running the ALTER SYSTEM command.
extra_desc | Can be set to off for environments where global configuration changes should be made using a different method.

```
<br/>
<br/>


There are two important things to keep in mind when using this new feature:
- **this is not a security feature**, it does not add any extra security layer;
- **`postgresql.auto.conf`** will be always loaded as last included file**, therefore setting the tunable to `off` will not change the configuration machinery of PostgreSQL, nor will make impossible for *external tools* to operate on `postgresql.auto.conf` directly (simulating, thefefore, `ALTER SYSTEM`),

Last but not least, keep in mind that the system is raising an error, thus aborting your existing scripts in the case this feature is set to `off`. According to me, the choice to error or ignoer an `ALTER SYSTEM` would have been a better choice, so that even automated script could rung without any side effect and without interruptions due to errors.
