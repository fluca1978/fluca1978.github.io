---
layout: post
title:  "GNU Guix and PostgreSQL"
author: Luca Ferrari
tags:
- guix
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Installing PostgreSQL via GNU Guix.

# GNU Guix and PostgreSQL

[GNU Guix](https://guix.gnu.org/en/download/){:target="_blank"} is **an advanced transactional package manager** for the GNU operating system. It is both a complete Linux distribution and a package manager that can be installed on an existing operating system.
<br/>
The idea behind GNU Guix is to provide a package manager that works in a way similar to that of *binary environment managers*: Guix uses *user profiles* and a set of self-contained directory tree to make available libraries and executables.
<br/>
GNU Guix provides a `guix` command line command that can be used to manage packages and all the GNU Guix dependencies and configuration.
<br/>
<br/>
In this article, I show how to use GNU Guix on a CentOS Linux operating system to install and manage PostgreSQL. Please note that I'm not going to show how to install `guix`, please refer to the [official GNU Guix installation guide](https://guix.gnu.org/manual/en/html_node/Binary-Installation.html){:target="_blank"}.

<br/>
<br/>
Please note that managing PostgreSQL versions via GNU Guix has nothing to do with PostgreSQL point in time recovery (PITR) or backup strategies.

# Using GNU Guix

The main command to interact with GNU Guix is, guess what, `guix`. The command allows for subcommands, in particular the `help` one that can provide you interactive help about other subcommands.
<br/>
In this article I'm going to use the following subcommands:
- `search` to search for packages to install;
- `pull` to get fresh `guix` package lists and update the program itself;
- `install` and `remove` to install a package and delete it;
- `package` the **main `guix` command**, many other subcommands are aliases to the `package` one. The `package` command allows for various operations on packages and their history.

## Searching for PostgreSQL

The subcommand `search` can be used to search for a package, in our case the beloved PostgreSQL database. The `search` command allows the specification of what to search as a regular expression, and in the following example I'll search for only packages that start with `postgresql` to avoid getting information about drivers and extensions:

<br/>
<br/>
```shell
% guix search '^postgresql.*$'

name: postgresql
version: 9.6.21
outputs: out
systems: x86_64-linux i686-linux
dependencies: openssl@1.1.1j readline@8.0.4 util-linux@2.35.1 zlib@1.2.11
location: \gnu/packages/databases.scm:1124:2\
homepage: https://www.postgresql.org/
license: X11-style
synopsis: Powerful object-relational database system  
description: PostgreSQL is a powerful object-relational database system.  It is fully ACID compliant, has full support for foreign keys, joins,
+ views, triggers, and stored procedures (in multiple languages).  It includes most SQL:2008 data types, including INTEGER, NUMERIC, BOOLEAN, CHAR,
+ VARCHAR, DATE, INTERVAL, and TIMESTAMP.  It also supports storage of binary large objects, including pictures, sounds, or video.
relevance: 30

name: postgresql
version: 13.2
outputs: out
systems: x86_64-linux i686-linux
dependencies: openssl@1.1.1j readline@8.0.4 util-linux@2.35.1 zlib@1.2.11
location: \gnu/packages/databases.scm:1085:2\
homepage: https://www.postgresql.org/
license: X11-style
synopsis: Powerful object-relational database system  
description: PostgreSQL is a powerful object-relational database system.  It is fully ACID compliant, has full support for foreign keys, joins,
+ views, triggers, and stored procedures (in multiple languages).  It includes most SQL:2008 data types, including INTEGER, NUMERIC, BOOLEAN, CHAR,
+ VARCHAR, DATE, INTERVAL, and TIMESTAMP.  It also supports storage of binary large objects, including pictures, sounds, or video.
relevance: 30
...
```
<br/>
<br/>
There are other PostgreSQL versions in the command output, that I trimmed out for sake of readibility. In short, the `search` allows you to search for a package and all its available versions.
<br/>
As many `guix` subcommands, the `search` command is just a shortcut for the invocation of the `package` subcommand with the appropriate options. In other words, `guix search foo` is the same as calling `guix package -s foo`:


<br/>
<br/>
```shell
% guix package -s '^postgresql.*$'

name: postgresql
version: 9.6.21
outputs: out
systems: x86_64-linux i686-linux
dependencies: openssl@1.1.1j readline@8.0.4 util-linux@2.35.1 zlib@1.2.11
location: \gnu/packages/databases.scm:1124:2\
homepage: https://www.postgresql.org/
license: X11-style
synopsis: Powerful object-relational database system  
description: PostgreSQL is a powerful object-relational database system.  It is fully ACID compliant, has full support for foreign keys, joins,
+ views, triggers, and stored procedures (in multiple languages).  It includes most SQL:2008 data types, including INTEGER, NUMERIC, BOOLEAN, CHAR,
+ VARCHAR, DATE, INTERVAL, and TIMESTAMP.  It also supports storage of binary large objects, including pictures, sounds, or video.
relevance: 30
...
```

<br/>
<br/>

## Installing PostgreSQL

If we don't specify any particular version, `guix` will install the latest available in its repositories, that as I write is PostgreSQL 13.2.
<br/>
There are two ways to install stuff in `guix`:
- compiling all software on the local machine (the default behaviour);
- using binary packages where and when available.

<br/>
Of course, compiling all the software on the local machine can require a lot of time and resources, depending on the power of the machine `guix` is running on.
<br/>
Binary packages are called **substitutes** in `guix`, because they *substitute source compiled software*.
<br/>
<br/>
In both installation scenarios, `guix` will install every dependency required by the specific software you are going to install.
<br/>
A source based installation will look like the following:


<br/>
<br/>
```shell
% guix install postgresql

The following package will be installed:
   postgresql 13.2
   
...
building /gnu/store/jkzin3sk1kk8ah9j066k3a03q4d99hc4-tcc-boot0-0.9.26-1103-g6e62e0e.drv...
| 'build' phase
building /gnu/store/35lsvpkqwgzmcs3gnhqkmxhivwfisidm-gzip-mesboot-1.2.4.drv...
building /gnu/store/gzlrw46slsi423qh5vcq91ki0rw4xzm4-make-mesboot0-3.80.drv...
building /gnu/store/2nvaxgs0rdxfkrwklh622ggaxg0wap6n-bash-mesboot0-2.05b.drv...
- 'unpack' phase
...

```
<br/>
<br/>

In the case of binary packages, *substitutions*, the installation will look like:

<br/>
<br/>
```shell
% guix install postgresql
...
 perl-5.30.2  13.6MiB                                                                                    1.6MiB/s 00:09 [##################] 100.0%
 pkg-config-0.29.2  201KiB                                                                               721KiB/s 00:00 [##################] 100.0%
 postgresql-13.2  5.4MiB                                                                                 543KiB/s 00:10 [##################] 100.0%
 guile-3.0.2  6.9MiB                                                                                     457KiB/s 00:15 [##################] 100.0%
 texinfo-6.7  1.2MiB                                                                                     3.5MiB/s 00:00 [##################] 100.0%
building CA certificate bundle...
building fonts directory...
building directory of Info manuals...
building database for manual pages...
building profile with 1 package...
hint: Consider setting the necessary environment variables by running:

     GUIX_PROFILE="/home/luca/.guix-profile"
     . "$GUIX_PROFILE/etc/profile"

Alternately, see `guix package --search-paths -p "/home/luca/.guix-profile"'.

```
<br/>
<br/>

The first time, `guix` has to *bootstrap* a lot of dependencies, so it will download, (build) and install libraries and tools even if they are already available on your operating system.
<br/>
At the end of the installation, `guix` will give you an hint about setting environment variables to give you access to the installed PostgreSQL (and other installed software).

<br/>
<br/>
Inspecting the content of the directory pointed by the above variable, you can see it contains PostgreSQL binaries and executables:

<br/>
<br/>
```shell
% export GUIX_PROFILE="/home/luca/.guix-profile"
% source "$GUIX_PROFILE/etc/profile"

% ls $GUIX_PROFILE  
bin  etc  include  lib  manifest  share

% ls /home/luca/.guix-profile/bin
clusterdb   dropuser  pg_archivecleanup  pg_config       pg_dumpall      pg_resetwal  pg_test_fsync    pg_waldump  reindexdb
createdb    ecpg      pg_basebackup      pg_controldata  pg_isready      pg_restore   pg_test_timing   postgres    vacuumdb
createuser  initdb    pgbench            pg_ctl          pg_receivewal   pg_rewind    pg_upgrade       postmaster  vacuumlo
dropdb      oid2name  pg_checksums       pg_dump         pg_recvlogical  pg_standby   pg_verifybackup  psql

```
<br/>
<br/>


### `locale` and Language problems

It is suggested to install the locales, because within `guix` ecosystem the ones you have already system-wide will not be available. This could make all your executables, included PostgreSQL's one, not working at all and cause you some problems.

<br/>
<br/>
```shell
% guix install glibc-locales

The following package will be installed:
   glibc-locales 2.31

...
 glibc-locales-2.31  10.8MiB                                                                             222KiB/s 00:50 [##################] 100.0%
 linux-libre-headers-5.4.20  1.0MiB                                                                      629KiB/s 00:02 [##################] 100.0%
building CA certificate bundle...
building fonts directory...
building directory of Info manuals...
building database for manual pages...
building profile with 3 packages...

```
<br/>
<br/>

After installing locales, you need to export Guix related environment variables to make the former available:

<br/>
<br/>
```shell
%  export GUIX_LOCPATH="$HOME/.guix-profile/lib/locale"
```
<br/>
<br/>


## Using the freshly installed PostgreSQL

Sourcing the `profile` file as suggested by `guix` makes the PostgreSQL executables available to your shell:

<br/>
<br/>
```shell
% which pg_ctl
~/.guix-profile/bin/pg_ctl
```
<br/>
<br/>

The trick is simple: the `profile` file manipulates the `PATH` environment variable to place the installed software executables in front of the already available ones:


<br/>
<br/>
```shell
% cat "$GUIX_PROFILE/etc/profile"

export PATH="${GUIX_PROFILE:-/gnu/store/xh9k8z9x5aspfqfcp1gycqlwksgl1m3g-profile}/bin${PATH:+:}$PATH"
```
<br/>
<br/>

It is now straightforward to use PostgreSQL as "usual":

<br/<
<br/>
```shell
% mkdir -p pgdata/13

% initdb -k -D pgdata/13
...
Success. You can now start the database server using:

    pg_ctl -D pgdata/13 -l logfile start

```
<br/>
<br/>

There is an important thing to note here: **PostgreSQL has been installed as a normal user**, this is very similar to virtual binary environment manages, for instance my favourite in PostgreSQL scenario [`pgenv`](https://github.com/theory/pgenv/).

<br/>
It is now possible to start PostgreSQL, and since I've already a system-wide PostgreSQL running, I need to specify a different port to listen on:

<br/>
<br/>
```shell
% pg_ctl -D pgdata/13 -o '-p 5433' start
waiting for server to start....
 LOG:  starting PostgreSQL 13.2 on x86_64-unknown-linux-gnu, compiled by gcc (GCC) 7.5.0, 64-bit
 LOG:  listening on IPv6 address "::1", port 5433
 LOG:  listening on IPv4 address "127.0.0.1", port 5433
 LOG:  listening on Unix socket "/tmp/.s.PGSQL.5433"
 LOG:  database system was shut down at 2021-09-30 08:41:44 EDT
 LOG:  database system is ready to accept connections
 done
server started

```
<br/>
<br/>

And it is now possible to see that two instances are running on the machine:

<br/>
<br/>
```shell
% psql -c 'SHOW SERVER_VERSION;' template1
 server_version 
----------------
 13.4
(1 row)

% psql -c 'SHOW SERVER_VERSION;' -p 5433 template1
 server_version 
----------------
 13.2
(1 row)

```
<br/>
<br/>

The PostgreSQL version 13.4 is the system wide one, while the version 13.2 is the one installed via `guix`.



## Getting Newer PostgreSQL Versions

In order to get newer PostgreSQL versions, you need to "ask" `guix` to search for updates. This is done via the `pull` command, that tell `guix` to update the list of available software:

<br/>
<br/>
```shell
% guix pull
Migrating profile generations to '/var/guix/profiles/per-user/luca'...
Updating channel 'guix' from Git repository at 'https://git.savannah.gnu.org/git/guix.git'...
Authenticating channel 'guix', commits 9edb3f6 to 7b59508 (6.374 new commits)...
Building from this channel:
  guix      https://git.savannah.gnu.org/git/guix.git   7b59508

...
 guix-7b59508ca-modules                                                                                        1.5MiB/s 00:20 | 29.2MiB transferred
 guix-module-union                                                                                                8.9MiB/s 00:00 | 3KiB transferred
 guix-command  635B                                                                                       18KiB/s 00:00 [##################] 100.0%
 guix-daemon  391B                                                                                       1.0MiB/s 00:00 [##################] 100.0%
 guix-7b59508ca                                                                                                 44.4MiB/s 00:00 | 16KiB transferred
building CA certificate bundle...
building fonts directory...
building directory of Info manuals...
building database for manual pages...
building profile with 1 package...
hint: Consider setting the necessary environment variables by running:

     GUIX_PROFILE="/home/luca/.config/guix/current"
     . "$GUIX_PROFILE/etc/profile"



% source "$GUIX_PROFILE/etc/profile"
``` 
<br/>
<br/>



**It is really important to `source` again the `profile` file** since it has changed due to the update process.

<br/>
After the `pull` update, we can search for PostgreSQL again and the available version has bumped to 13.3:


<br/>
<br/>
```shell
% guix search 'postgresql.*'
...
name: postgresql
version: 13.3
outputs: out
systems: x86_64-linux i686-linux
dependencies: openssl@1.1.1j readline@8.0.4 util-linux@2.35.1 zlib@1.2.11
location: \gnu/packages/databases.scm:1127:2\
homepage: https://www.postgresql.org/
license: X11-style
synopsis: Powerful object-relational database system  
description: PostgreSQL is a powerful object-relational database system.  It is fully ACID compliant, has full support for foreign keys, joins,
+ views, triggers, and stored procedures (in multiple languages).  It includes most SQL:2008 data types, including INTEGER, NUMERIC, BOOLEAN, CHAR,
+ VARCHAR, DATE, INTERVAL, and TIMESTAMP.  It also supports storage of binary large objects, including pictures, sounds, or video.
relevance: 30
...
```
<br/>
<br/>

It is now time to upgrade the currently running PostgreSQL (and it is suggested to stop the running instance before);

<br/>
<br/>
```shell
% pg_ctl -D pgdata/13 stop

% guix upgrade postgresql
The following package will be upgraded:
   postgresql 13.2 → 13.3

...
 postgresql-13.3  5.4MiB                                                                                 1.7MiB/s 00:03 [##################] 100.0%
substitute: updating substitutes from 'https://ci.guix.gnu.org'... 100.0%
The following derivation will be built:
   /gnu/store/ghlc1angdx9q7gx4hm4yagam6m0gmxzw-profile.drv

0,2 MB will be downloaded
...

```

<br/>
<br/>

Is the new PostgreSQL version installed? Let's check out:

<br/>
<br/>
```shell
% pg_ctl -D pgdata/13 -o '-p 5433' start
...
server started

% psql -p 5433 -c 'SHOW SERVER_VERSION;' template1
 server_version 
----------------
 13.3
(1 row)

```
<br/>
<br/>
Success!


## Generations (or, "How do I go back in time?")

`guix` stores the so called **generations**, that are *point in time* that contain the history of the installed/removed packages.
The `package` sub command can show you the generations available in your system, for example:


<br/>
<br/>
```shell
% guix package --list-generations
\Generation 1   set 30 2021 08:16:02\
  postgresql    13.2    out     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2

\Generation 2   set 30 2021 08:37:18\
 + glibc-utf8-locales   2.31    out     /gnu/store/rgydar9dfvflqqz2irgh7njj34amaxc6-glibc-utf8-locales-2.31

\Generation 3   set 30 2021 08:40:43\
 + glibc-locales        2.31    out     /gnu/store/wnw0nwlyg92vv33f5f65jj1rd3p4fi3c-glibc-locales-2.31

\Generation 4   set 30 2021 10:04:21\   (current)
 + postgresql   13.3    out     /gnu/store/1nlzmg4hw4gga56g58dsqf9nx90z9kkn-postgresql-13.3
 - postgresql   13.2    out     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2

```
<br/>
<br/>

In the above example, we installed PostgreSQL 13.2 as first thing (*generation 1*), while the upgrade of PostgreSQL to version 13.3 happened in the fourth generation. Note that the output is somehow similar to a `diff` status report, where `+` lines are addition and `-` are somehow removals.
<br/>
Imagine we need to come back to version 13.2 of PostgreSQL. How can we achieve this?
There are two ways:
- do a so called *rollback* that makes the last generation active (that is goes to generation number 3);
- jump to a specific revision, in this case the number 1.

<br/>
Depending on the history of your system, you can choose the correct approach.
<br/>
Let's jump to generation one (again, ensure your PostgreSQL server is turned off):

<br/>
<br/>
```shell
% pg_ctl -D pgdata/13 stop

% guix package --switch-generation=1
switched from generation 4 to 1

% pg_ctl --version
pg_ctl (PostgreSQL) 13.2
```
<br/>
<br/>

Unlike installing new software, switching to a previous generation is a very fast, almost immediate, operation, since the only thing to do is to adjust the binary environment. As you can see, the PostgreSQL executables are turned back to version 13.2.

<br/>
<br/>
What if we want to upgrade PostgreSQL version again? One solution is to `switch-generation` again, but it is also possible to run `upgrade` again, that is an almost immediate operation since everything is already on the system:


<br/>
<br/>
```shell
% guix upgrade postgresql

building CA certificate bundle...
listing Emacs sub-directories...
building fonts directory...
building directory of Info manuals...
building database for manual pages...
building profile with 1 package...

% pg_ctl --version
pg_ctl (PostgreSQL) 13.3
```
<br/>
<br/>

What has changed in the *generations*? Since we moved back to history placeholder *one*, and then upgrade PostgreSQL, the upgrade has been *squashed* from there:

<br/>
<br/>
```shell
% guix package --list-generations

\Generation 1   Sep 30 2021 08:16:02\
  postgresql    13.2    out     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2

\Generation 2   Sep 30 2021 10:15:33\   (current)
 + postgresql   13.3    out     /gnu/store/1nlzmg4hw4gga56g58dsqf9nx90z9kkn-postgresql-13.3
 - postgresql   13.2    out     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2

\Generation 3   Sep 30 2021 08:40:43\
 + glibc-locales        2.31    out     /gnu/store/wnw0nwlyg92vv33f5f65jj1rd3p4fi3c-glibc-locales-2.31
 + glibc-utf8-locales   2.31    out     /gnu/store/rgydar9dfvflqqz2irgh7njj34amaxc6-glibc-utf8-locales-2.31
 + postgresql           13.2    out     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2
 - postgresql           13.3    out     /gnu/store/1nlzmg4hw4gga56g58dsqf9nx90z9kkn-postgresql-13.3

\Generation 4   Sep 30 2021 10:04:21\
 + postgresql   13.3    out     /gnu/store/1nlzmg4hw4gga56g58dsqf9nx90z9kkn-postgresql-13.3
 - postgresql   13.2    out     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2

```
<br/>
<br/>

The PostgreSQL set of changes is propagated from history number one to all the other entries.



## Removing PostgreSQL

Imagine we want to remove the 13.3 PostgreSQL version, keeping the older one available. The `remove` command does pretty much what you would expect:


<br/>
<br/>
```shell
% guix remove postgresql@13.3

The following package will be removed:
   postgresql 13.3

The following derivation will be built:
   /gnu/store/ilxkw0i597n0qvirb11mksbyad8qmnvd-profile.drv

building profile with 0 packages...

```
<br/>
<br/>

Again, this is a very fast operation, and this should hint you that nothing has been removed from the storage. Note that I specified the version to remove with the `@<version>` syntax after the package name.
<br/>
Is the older PostgreSQL version immediatly available? NO!
<br/>
If you test the binaries, you will find out that the system wide (if any) because, as `guix` has told you in the above command output, it has removed the package from the current profile. This means, PostgreSQL is no more available via `guix`:


<br/>
<br/>
```shell
% which pg_ctl
/usr/pgsql-13/bin/pg_ctl

% pg_ctl --version
pg_ctl (PostgreSQL) 13.4
```
<br/>
<br/>

In order to enable the PostgreSQL 13.2 version via `guix package --switch-generations` to jump back to the generation that has the required PostgreSQL package:


<br/>
<br/>
```shell
% guix package --switch-generation=1
switched from generation 3 to 1

% pg_ctl --version
pg_ctl (PostgreSQL) 13.2

```
<br/>
<br/>

But how to free disk space from unused PostgreSQL versions?
<br/>
The `gc` subcommand will *garbage collect* packages that are not in use. **Here, *not in use* could be something different from what you think: `guix` is of course smarter than you (at least, smarter than me) in finding out references between generations and packages**. This means that the only fact that a package is not currently in use, does not make it eligible for hard deletion. It is therefore recommended to delete all the generations that refer to a specific package in order to get it deleted from the garbage collector:

<br/>
<br/>
```shell
% guix package --delete-generations=2
% guix package --delete-generations=3
% guix package --delete-generations=4
% guix gc
...
note: currently hard linking saves 913.85 MiB
guix gc: freed 3,631.25278 MiBs

% du -hs /gnu/store/*postgresql-13.?    
30M     /gnu/store/ivmkwkjsvbkv3g0jq9gcgwlhrhwx91gw-postgresql-13.2
```
<br/>
<br/>

Of course, this approach brings back your *whole* system since upgrading will require a new fresh installtion.



# Conclusions

GNU Guix is a very interesting package manager that can be used to setup a binary environment useful for testing and deploying software stacks, including our beloved database and its dependencies (e.g., tools and libraries).
<br/>
Probably you are not going to use `guix` in a PostgreSQL production environment because you will have other package revision tools, to automate and keep *stable* your packages. However, `guix` can be very handy in testing and upgrading your own environment.
