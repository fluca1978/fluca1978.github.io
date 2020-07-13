---
layout: post
title:  "PostgreSQL 13 Beta 2: it's your time to help testing!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv

permalink: /:year/:month/:day/:title.html
---
We are approaching the great 13 release, help the team testing it!

# PostgreSQL 13 Beta 2: it's your time to help testing!

We are approaching very quickly (and on time) the *[PostgreSQL 13](https://www.postgresql.org/about/news/2047/){:target="_blank"}* version, and **we all can help testing it** to provide a feedback and get ready for the next version.
<br/>
As I've often written, a very easy approach to install and test the new version along side the version you are using (but not in production!) is by means of [pgenv](https://github.com/theory/pgenv){:target="_blank"}.
<br/>
The only thing you have to do is `pgenv build 13beta2`, or if you are more curious:

```shell
luca@miguel ~ % pgenv available 13
             Available PostgreSQL Versions
========================================================

                     PostgreSQL 13
    ------------------------------------------------
     13beta1  13beta2 

luca@miguel ~ % pgenv build 13beta2
...
PostgreSQL, contrib, and documentation installation complete.
pgenv configuration written to file /home/luca/git/pgenv/.pgenv.13beta2.conf
PostgreSQL 13beta2 built
```

<br/>
<br/>
Once the system has been compiled, you can start it and use it:
<br/>

```shell
luca@miguel ~ % pgenv use 13beta2           

WARNING:
  your PATH enrvironemnt variable does not seem to include

       /home/luca/git/pgenv/pgsql/bin

  as an entry. You will not be able to use the currently
  selected PostgreSQL binaries.

HINT:
  adjust your PATH variable to include

  /home/luca/git/pgenv/pgsql/bin

  for instance

  export PATH=/home/luca/git/pgenv/pgsql/bin:$PATH

Already using PostgreSQL 13beta2
waiting for server to start.... done
server started
PostgreSQL 13beta2 started
Logging to /home/luca/git/pgenv/pgsql/data/server.log
```

<br/>
As the `pgenv` output suggests, it is better to modify your `PATH` to get the new executables:

<br/>
```shell
luca@miguel ~ % export PATH=/home/luca/git/pgenv/pgsql/bin:$PATH
```

<br/>
and you can make this a permanent change in your shell configuration.
<br/>
<br/>
Now, let's connect to the cluster:

<br/>
<br/>
```shell
luca@miguel ~ % psql -U postgres template1 -c 'SHOW server_version;'
 server_version 
----------------
 13beta2
(1 row)

luca@miguel ~ % psql -U postgres template1 -c 'SELECT version();'   
                                                  version                                                   
------------------------------------------------------------------------------------------------------------
 PostgreSQL 13beta2 on x86_64-unknown-freebsd12.1, compiled by gcc (FreeBSD Ports Collection) 9.2.0, 64-bit
(1 row)

```

<br/>
Happy testing!
