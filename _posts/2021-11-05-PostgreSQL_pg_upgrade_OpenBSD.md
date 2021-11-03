---
layout: post
title:  "pg_upgrade and OpenBSD"
author: Luca Ferrari
tags:
- openbsd
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
OpenBSD ships `pg_upgrade` as a separate package.

# pg_upgrade and OpenBSD

I never noted that, on OpenBSD, the `pg_upgrade` command is not shipped with the *default* PostgreSQL server isntallation. I usually install PostgreSQL from sources, so I never digged into Open BSD packages. The choice of OpenBSD is to keep `pg_upgrade` separate from the rest of the binaries and executables of PostgreSQL.
<br/>
Allow me to explain and let's start from the installed binaries on a OpenBSD 7.0 machine:

<br/>
<br/>
```shell
% ls -1 /usr/local/bin/pg*
/usr/local/bin/pg_archivecleanup
/usr/local/bin/pg_basebackup
/usr/local/bin/pg_checksums
/usr/local/bin/pg_config
/usr/local/bin/pg_controldata
/usr/local/bin/pg_ctl
/usr/local/bin/pg_dump
/usr/local/bin/pg_dumpall
/usr/local/bin/pg_isready
/usr/local/bin/pg_receivewal
/usr/local/bin/pg_recvlogical
/usr/local/bin/pg_resetwal
/usr/local/bin/pg_restore
/usr/local/bin/pg_rewind
/usr/local/bin/pg_standby
/usr/local/bin/pg_test_fsync
/usr/local/bin/pg_test_timing
/usr/local/bin/pg_verifybackup
/usr/local/bin/pg_waldump
/usr/local/bin/pgbench

```
<br/>
<br/>

The server is a PostgreSQL 13.4, installed via `pkg_add`. The PostgreSQL *contrib* module is installed, but as you can see, there is no `pg_upgrade` binary in the above listing.
<br/>
Let's inspect the packages:

<br/>
<br/>
```shell
% pkg_info -Q postgresql

postgresql-client-13.4p0 (installed)
postgresql-contrib-13.4p0 (installed)
postgresql-docs-13.4p0 (installed)
postgresql-odbc-10.02.0000p0
postgresql-pg_upgrade-13.4p0
postgresql-pllua-2.0.7
postgresql-plpython-13.4p0
postgresql-plr-8.4.1
postgresql-server-13.4p0 (installed)
```
<br/>
<br/>

Please note the `postgresql-pg_upgrade-13.4p0` that is what contains the `pg_upgrade` command:


<br/>
<br/>
```shell
% pkg_info  postgresql-pg_upgrade-13.4p0 
Information for https://cdn.openbsd.org/pub/OpenBSD/7.0/packages/amd64/postgresql-pg_upgrade-13.4p0.tgz

Comment:
Support for upgrading PostgreSQL data from previous version

Description:
Contains pg_upgrade, used for upgrading PostgreSQL database
directories to newer major versions without requiring a dump and
reload.

Maintainer: Pierre-Emmanuel Andre <pea@openbsd.org>

WWW: https://www.postgresql.org/

```
<br/>
<br/>

This choice of packaging is somehow strange.
<br/>
Let's install `pg_upgrade`:


<br/>
<br/>
```shell
% doas pkg_add postgresql-pg_upgrade
quirks-4.53 signed on 2021-10-30T11:32:24Z
postgresql-pg_upgrade-13.4p0:postgresql-previous-12.8: ok
postgresql-pg_upgrade-13.4p0: ok

% ls -lh $(which pg_upgrade)
-rwxr-xr-x  1 root  bin   185K Sep 26 21:25 /usr/local/bin/pg_upgrade
```
<br/>
<br/>

So, the binary itself is very tiny, and sizes at `185 kB`, therefore placing it on its own package does not make sense with regard to the disk space occupation. However, please note that installing `pg_upgrade` also triggered the installation of `postgresql-previous-12.8`, that means the system has installed also PostgreSQL 12.8.
<br/>
This is clearly shown from a query on such package:

<br/>
<br/>
```shell
% pkg_info postgresql-previous-12.8   
Information for inst:postgresql-previous-12.8

Comment:
PostgreSQL RDBMS (previous version, for pg_upgrade)

Required by:
postgresql-pg_upgrade-13.4p0

Description:
PostgreSQL RDBMS server, the previous version

This is the previous version of PostgreSQL, necessary to allow for
pg_upgrade to work in the currently supported PostgreSQL version.

```
<br/>
<br/>

And in fact, the package installs *all* the previous version of the cluster, included libraries and executables:

<br/>
<br/>
```shell
% pkg_info -L postgresql-previous-12.8 | grep bin
/usr/local/bin/postgresql-12/clusterdb
/usr/local/bin/postgresql-12/createdb
/usr/local/bin/postgresql-12/createuser
/usr/local/bin/postgresql-12/dropdb
/usr/local/bin/postgresql-12/dropuser
/usr/local/bin/postgresql-12/ecpg
/usr/local/bin/postgresql-12/initdb
/usr/local/bin/postgresql-12/oid2name
/usr/local/bin/postgresql-12/pg_archivecleanup
/usr/local/bin/postgresql-12/pg_basebackup
/usr/local/bin/postgresql-12/pg_checksums
/usr/local/bin/postgresql-12/pg_config
...
```
<br/>
<br/>

**Therefore, installing `pg_upgrade` will also install the *whole** previous major version of PostgreSQL.** 

## It was a separated packages since a while...

Inspecting the CVS of the ports tree, it is possible to note that the [`pg_upgrade` command has been separated into a *subpackage* since 2016](https://cvsweb.openbsd.org/cgi-bin/cvsweb/ports/databases/postgresql/pkg/DESCR-pg_upgrade?rev=1.1&content-type=text/x-cvsweb-markup){:target="_blank"}:

<br/>
<br/>
```
This moves pg_upgrade to a subpackage, and has that
subpackage depend on postgresql-previous.
```
<br/>
<br/>

In fact, [this is the commit](https://cvsweb.openbsd.org/cgi-bin/cvsweb/ports/databases/postgresql/Makefile.diff?r1=1.218&r2=1.219&f=h){:target="_blank"} that made the `pg_upgrade` a distinct package into the build system.
<br/>
The rationale about this can be found [in the b2k16 hackaton article](https://undeadly.org/cgi?action=article;sid=20161112112023){:target="_blank"}, where Jeremy Evans explain that in order to get `pg_upgrade` to work, there was the need to have the previous binaries for PostgreSQL. Therefore, the application has been moved to a different package, so that it can install also the previous binaries on the system.

# Conclusions

The choice of keeping `pg_upgrade` as a separated package is a choice. I don't think it is right or wrong, it is just a choice that ensures that if you decide to install a newer PostgreSQL, you must have a previous version to upgrade from. 
<br/>
Quite frankly, I don't see the reason because I could have a different database version into the system, that I want to upgrade from, even if I did not have installed from ports.
<br/>
Moreover, `pg_upgrade` can upgrade PostgreSQL even from non-sequential PostgreSQL versions, even if I personally don't recommend this, especially if the "hole" in versioning is big. However, this means that installing the previous version of PostgreSQL could not be the right choice in every scenario. Again, *this is not either a good or bad choice, it is just a choice* and it must be noted that, unlike other operating systems, OpenBSD does not offer old versions of PostgreSQL as packages (if we exclude the `-previous` package), that means it is a choice coherent with the philosophy of the operating system.
