---
layout: post
title:  "SQLite3 Vacuum and Autovacuum"
author: Luca Ferrari
tags:
- sqlite
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Similarly to PostgreSQL, also SQLite3 needs some care...

# SQLite3 Vacuum and Autovacuum

Today I discovered, by accident I need to confess, that PostgreSQL is not the only database requiring `VACUUM`: also SQLite3 does.

And there's more: SQLite3 includes an *auto-vacuum* too!
They behave similarly, at least in theory, to their PostgreSQL counterparts, but clearly there is no autovacuum daemon or process. Moreover, the configuration is simpler and I've not found any threshold as we have in PostgreSQL. In the following, I explain how `VACUUM` works in SQLite3, at least at glance.

SQLite3 does not have a fully enterprise-level MVCC machinery as PostgreSQL has, but when tuples or tables are updated or deleted from a database, defragmentation and not reclaimed space makes the database file never shrink.
Similarly to what PostgreSQL does, the *now* empty space (no more occupied by old tuples) is kept for future usage, so that the effect is that the database grows without never shrinking even after large data removal.

`VACUUM` is the solution that also SQLite3 uses to reclaim space.

[VACUUM](https://sqlite.org/lang_vacuum.html){:target="_blank"} is a command available to the SQLite3 prompt to start a manual space reclaiming. It works by copying the database file content into another (temporary) file and restructuring it, so nothing really fancy and new here!

Then comes [auto-vacuum](https://sqlite.org/pragma.html#pragma_auto_vacuum){:target="_blank"} that is turned off by default. The autovacuum works in a *full* mode or an *incremental mode*. The former is the most aggressive, and happens after a `COMMIT`. The second is the less intrusive, and "prepares" what the vacuum process has to do, without performing it. Is is only when `[incremental_autovacuum](https://sqlite.org/pragma.html#pragma_incremental_vacuum){:target="_blank"}` is launched that the space is freed. Therefore, autovacuum is SQLite3 either executes at each `COMMIT` or is postponed when considered safe to execute.
