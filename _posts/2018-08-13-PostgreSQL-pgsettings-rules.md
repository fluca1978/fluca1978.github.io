---
layout: post
title:  "An example of PostgreSQL rules: updating pg_settings"
author: Luca Ferrari
tags:
- postgresql
- itpug
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Rules are a powerful mechanism by which PostgreSQL allows a statement to be *transformed* into another.
And PostgreSQL itself does use rules in order to make your life easier.

# An example of PostgreSQL rules: updating pg_settings

When asked for a quick and sweet example about rules I often answer with the `pg_settings` example.

The special view `pg_settings` offers a tabular decodification of the current cluster settings, in other words allows you to see `postgresql.conf` (and friends) as  a table to run queries against.

But there is more than that: you can also issue `UPDATE` commands against such table and get the configuration updated on the fly (this does not mean *applied*, it depends on the parameter context). Internally, PostgreSQL uses a very simple **rule** to cascade updates to `pg_settings` into the run-time configuration. The rule can be found in the `system_views.sql` files inside the backend source code and is implemented as:


```sql
CREATE RULE pg_settings_u AS
    ON UPDATE TO pg_settings
    WHERE new.name = old.name DO
    SELECT set_config(old.name, new.setting, 'f');
```

It simply reads as: whenever there is an update keeping untouched the parameter name, invoke the special function `set_config` with the parameter name and its new value (the flag `f` means to keep changes not *local* to session). For more information about `set_config` see [the function official documentation](https://www.postgresql.org/docs/current/static/functions-admin.html).

How cool!
