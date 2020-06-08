---
layout: post
title:  "Locating the PostgreSQL configuration file"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- docker
permalink: /:year/:month/:day/:title.html
---
How to find the PostgreSQL configuration file on an unknown system?

# Locating the PostgreSQL configuration file

Sometimes you get to manage a PostgreSQL instance on an *unknown* system, and this means you don't know how to locate the PostgreSQL configuration file.
<br/>
An example could be when you are running PostgreSQL on a Docker container:


```shell
root@ff20ff72ee64:/# ps -auxwww
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
postgres     1  0.0  0.8 288596 18076 ?        Ss   16:05   0:00 postgres
postgres    22  0.0  0.1 288732  3860 ?        Ss   16:05   0:00 postgres: checkpointer  
postgres    23  0.0  0.1 288596  3096 ?        Ss   16:05   0:00 postgres: background writer  
postgres    24  0.0  0.3 288596  6260 ?        Ss   16:05   0:00 postgres: walwriter  
postgres    25  0.0  0.1 289024  3068 ?        Ss   16:05   0:00 postgres: autovacuum launcher  
postgres    26  0.0  0.1 143856  2264 ?        Ss   16:05   0:00 postgres: stats collector  
postgres    27  0.0  0.1 288884  2564 ?        Ss   16:05   0:00 postgres: logical replication launcher  
root        70  0.0  0.1  19856  2236 pts/0    Ss   16:12   0:00 /bin/bash
```

<br/>
Assuming you have the credentials of a PostgreSQL *super user*, you can ask PostgreSQL itself:

```shell
root@ff20ff72ee64:/# psql -U postgres -c 'SHOW config_file'        
               config_file                
------------------------------------------
 /var/lib/postgresql/data/postgresql.conf
(1 row)

```

> You should have the administrator user credentials, since you have been assigned to manage this PostgreSQL instance!

Of course, you can have your credentials stored into the `.pgpass` file, as usual.
<br/>
You can also save some extra "attaching" operations by macking `docker` do all the stuff for you:


```shell
$ docker exec -it db psql -U ckan -c 'SHOW config_file'
               config_file                
------------------------------------------
 /var/lib/postgresql/data/postgresql.conf
(1 row)
```


<br/>
There are other methods, of course, including a search with `find(1)` or `locate`.

