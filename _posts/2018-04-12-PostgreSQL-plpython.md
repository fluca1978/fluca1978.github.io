---
layout: post
title:  "PostgreSQL 10 and Python 3 (on FreeBSD)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- python
permalink: /:year/:month/:day/:title.html
---
Today a friend of mine asked me for a trouble between PostgreSQL 10 and Python. Since I'm not a pythonist, my quick answer was that PostgreSQL 10 does support Python 3. But it turned out it was not so simple (at least, not so simple as to get Perl 5 working!).

# PostgreSQL 10, Python 3 and FreeBSD (and Ubuntu)

tl;dr *it can be done* (of course!)

```sql
postgres=# SELECT pyv();
                                    pyv                                     
----------------------------------------------------------------------------
 3.6.4 (default, Jan  2 2018, 01:25:35)                                    +
 [GCC 4.2.1 Compatible FreeBSD Clang 4.0.0 (tags/RELEASE_400/final 297347)]
 
postgres=# SELECT proname, prolang, prosrc FROM pg_proc WHERE proname = 'pyv';
-[ RECORD 1 ]-----------------
proname | pyv
prolang | 16387
prosrc  |                     +
        |   import sys        +
        |   return sys.version+
        | 
```


My default environment for running PostgreSQL is FreeBSD, so in order to get `plpython` working I jumped to the console and installed the package `postgresql10-plpython-10.3`, thinking of course that FreeBSD would do the right thing. Unluckily it did not!

Creating a `plpython3u` language did not succeed, and the problem was that the above package installed only the `libpython2.so` under the `lib` directory (e.g., `/usr/local/lib/postgresql`). I then tried installing the port of the very same name, but again it was installing only the `libpython2.so`.

In order to better understand what was missing, I switched to a clean /Ubuntu 17.10/ installing [all the `deb` packages for PostgreSQL and Python](https://www.postgresql.org/download/linux/ubuntu/), but again this was not working. The problem seemed to me that the system was using Python 3.7 and the `plpython` package was requiring Python 3.5, which is no more supported on that version of Ubuntu.
I then switched back to another Ubuntu machine, older than the previous one, and tried installing the whole [Enterprise DB Interactive Installer](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads), hoping it would come with a self-contained version. Again, this was not working, since the server was unable to load the `libpython3.so` *even if that was in place*!
I then inspected such file in order to see what it was missing:

```sh
$ ldd /opt/PostgreSQL/10/lib/postgresql/plpython3.so
        linux-vdso.so.1 =>  (0x00007ffef10fd000)
        libpython3.4m.so.1.0 => not found
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fbf394cb000)
        /lib64/ld-linux-x86-64.so.2 (0x000055d03b80f000)

```

Fine: `libpython3.4m.so` is missing. I then searched my hard drive and found a version laying around, therefore exported `LD_LIBRARY_PATH` to include the path to such file and restarted the server.
This time I was able to create the language `plpython3u`.
But when I tried to define the  above `pyv` function, the backend process crashed!
Inspecting the log I found:

```sh
Could not find platform independent libraries <prefix>
Could not find platform dependent libraries <exec_prefix>
Consider setting $PYTHONHOME to <prefix>[:<exec_prefix>]
Fatal Python error: Py_Initialize: Unable to get the locale encoding
ImportError: No module named 'encodings'
LOG:  server process (PID 18776) was terminated by signal 6: Aborted
```

So, it seemed a Python-configuration problem. However I was unable to solve it, probably due to my poor Python knowledge.

I then switched back to FreeBSD and got the latest PostgreSQL 10.3 source code.
I then compiled it passing the `--with-python` at `configure` time and after a while I had a PostgreSQL 10.3 instance running and with Python 3.6 working.

Therefore, at the end of this story, I was unable to get Python 3 working within the `plpython` language until I compile PostgreSQL by myself. I suspect binary packages should advice the lack of such language support to avoid users' troubles. Of course, this could have just been due to my poor knowledge of Python itself...
