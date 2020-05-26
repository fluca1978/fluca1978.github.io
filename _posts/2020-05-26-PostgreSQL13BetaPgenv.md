---
layout: post
title:  "PostgreSQL 13 beta 1 on FreeBSD via pgenv"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- freebsd
permalink: /:year/:month/:day/:title.html
---
It's time to test the new PostgreSQL 13 release!

# PostgreSQL 13 beta 1 on FreeBSD via pgenv

Five days ago [PostgreSQL 13 beta 1](https://www.postgresql.org/about/news/2040/){:target="_blank"} has been released!
<br/>
It's time to test the new awesome version of our beloved database. Installing from source is quite trivial, but why not using [pgenv](https://github.com/theory/pgenv){:target="_blank"} to such aim?
<br/>
Installing on my FreeBSD machine with `pgenv` is as simple as:

```shell
luca@miguel ~ % pgenv build 13beta1
...
PostgreSQL, contrib, and documentation installation complete.
pgenv configuration written to file /home/luca/git/pgenv/.pgenv.13beta1.conf
PostgreSQL 13beta1 built
```

<br/>
Are you ready to test it?
<br/>
Activate it and enjoy:

```shell
luca@miguel ~ % pgenv use 13beta1
...
server started
PostgreSQL 13beta1 started
Logging to /home/luca/git/pgenv/pgsql/data/server.log


luca@miguel ~ % psql -U postgres -c "SELECT version();" template1
                                                  version                                                   
------------------------------------------------------------------------------------------------------------
 PostgreSQL 13beta1 on x86_64-unknown-freebsd12.1, compiled by gcc (FreeBSD Ports Collection) 9.2.0, 64-bit
(1 row)
```

*Enjoy!*
