---
layout: post
title:  "Installing PostgreSQL on OpenBSD" 
author: Luca Ferrari
tags:
- openbsd
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A quick look at how to get PostgreSQL up and running on OpenBSD.

# Installing PostgreSQL on OpenBSD

OpenBSD is a rock solid, super secure, real Unix operating system. 
<br/>
PostgreSQL is a rock solid, enterprise level, fully feautured relational database.
<br/>
Is it possible to merge the two for a great database on such operating system? *Yes*, of course!
<br/>
<br/>
OpenBSD has a packaging system that is somehow different from many other operating systems; in particular the packages are deeply inspected before they are installed, so that the installation process proceed only if it really sure the installation can succeed. Moreover, the operating system provides a simple and flexible way to manage services like PostgreSQL.
<br/>
In this short article, I will show how you can start working with PostgreSQL on OpenBSD.

## Packages

OpenBSD uses the `pkg_xxx` tools, a set of intelligent Perl applications that handle all the packaging mechanics. While it is true that you can install an application out of *ports*, like other BSDs, OpenBSD recommends to install via packages because the system can easily track what you have installed so far, and consequently, handle updates.
<br/>
<br/>
The first thing to do is therefore to search for some PostgreSQL related package, and this is done by means of `pkg_info` command, with the particular `-Q` flag (for "query"):

<br/>
<br/>
```shell
puffy# pkg_info -Q postgresql
debug-dovecot-postgresql-2.3.15v0
debug-dovecot-postgresql-2.3.16v0
dovecot-postgresql-2.3.15v0
dovecot-postgresql-2.3.16v0
postgresql-client-13.4
postgresql-contrib-13.4
postgresql-docs-13.4
postgresql-pg_upgrade-13.4
postgresql-plpython-13.4
postgresql-previous-12.8
postgresql-server-13.4

```
<br/>
<br/>

As you can see, the currently supported version of PostgreSQL is 13.4, while the 14 is already out from a few days (at the time of writing).
<br/>
Installing packages is done by means of `pkg_add`, and this case there is no particular *flavour* (i.e., configuration or stack) required, so it is as simple as:

<br/>
<br/>
```shell
puffy# pkg_add postgresql-server-13.4 postgresql-client-13.4 postgresql-contrib-13.4 postgresql-docs-13.4
quirks-3.633 signed on 2021-10-05T18:48:49Z
postgresql-server-13.4:libexecinfo-0.3p2v0: ok
postgresql-server-13.4:xz-5.2.5: ok
postgresql-server-13.4:libiconv-1.16p0: ok
postgresql-server-13.4:libxml-2.9.10p3: ok
postgresql-server-13.4:postgresql-client-13.4: ok
useradd: Warning: home directory `/var/postgresql' doesn't exist, and -m was not specified
postgresql-server-13.4: ok
postgresql-contrib-13.4: ok
postgresql-docs-13.4: ok
Running tags: ok
The following new rcscripts were installed: /etc/rc.d/postgresql
See rcctl(8) for details.
New and changed readme(s):
        /usr/local/share/doc/pkg-readmes/postgresql-server

```
<br/>
<br/>

It takes less than a minute to have all the components installed (and if you are curious, it takes much more time to install Emacs 27.2 without X Window support!).
<br/>
It is important to note that a new *rc-script* has been installed. *rc-scripts* are a set of well defined Korn Shell based scripts that are used to manage daemons; they act similar to other init systems like *systemd* without being, well, so much bloated.
<br/>
Since the installed script is named `/etc/rc.d/postgresql`, the service will be called as the relative file name of the script, therefore `postgresql`.


## Start the PostgreSQL Server (and Failing)

You will not be able to start PostgreSQL just after the installation:

<br/>
<br/>
```shell
puffy# rcctl start postgresql
postgresql(failed)
```
<br/>
<br/>

To understand why PostgreSQL is not starting, we need to dig a little more into the *rc-scripts*. First of all, ask the OpenBSD system what it knows about PostgreSQL, and this is done thru the `rcctl` command and the `get` option:

<br/>
<br/>
```shell
puffy# rcctl get postgresql
postgresql_class=daemon
postgresql_flags=NO
postgresql_logger=
postgresql_rtable=0
postgresql_timeout=30
postgresql_user=_postgresql

```
<br/>
<br/>

There is not much output in the above command, but essentially PostgreSQL is system-wide disabled. However, this is not why the process is failing, and in order to discover what is causing the fault, we need to *debug* the `rcctl` execution:

<br/>
<br/>
```shell
puffy# rcctl -df start postgresql 
doing _rc_parse_conf
doing _rc_quirks
postgresql_flags empty, using default >-D /var/postgresql/data -w -l /var/postgresql/logfile<
doing rc_check
pg_ctl: directory "/var/postgresql/data" does not exist
postgresql
doing rc_start
doing _rc_wait start
doing rc_check
pg_ctl: directory "/var/postgresql/data" does not exist
pg_ctl: directory "/var/postgresql/data" does not exist
doing _rc_rm_runfile
(failed)

```
<br/>
<br/>

Essentially, the system is failing because **packages did not created the PGDATA directory**, and therefore this must be done manually:

<br/>
<br/>
```shell
puffy# mkdir /var/postgresql/data
puffy# chown _postgresql:_postgresql /var/postgresql/data

puffy# su - _postgresql

puffy$ initdb -D /var/postgresql/data
The files belonging to this database system will be owned by user "_postgresql".
This user must also own the server process.

The database cluster will be initialized with locale "C".
The default database encoding has accordingly been set to "SQL_ASCII".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/postgresql/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 20
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Rome
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D /var/postgresql/data -l logfile start


```
<br/>
<br/>

Now that the data directory is set up, the system can be started:

<br/>
<br/>
```shell
puffy# rcctl start postgresql
postgresql(ok)
```
<br/>
<br/>

Of course, you can play around the fresh installed and fired up PostgreSQL:

<br/>
<br/>
```shell
puffy# psql -U _postgresql template1
psql (13.4)
Type "help" for help.

template1=# SHOW SERVER_VERSION;
 server_version 
----------------
 13.4
(1 row)

```
<BR/>
<BR/>

Did you spot the little trick up there? **Since the `initdb` has been executed by the `_postgresql` user, the database administrator is the `_postgresql` user too!**


## Configuring your PostgreSQL, the OpenBSD way!

The `rcctl` set of scripts is based on a set of variables, that can construct a set of flags passed to the daemon in order to configure it. For example, in the default installation, the `PGDATA` directory is set to `/var/postgresql/data`, but where is this set? Let's inspect again what the rc-scripts knows about:


<br/>
<br/>
```shell
puffy# rcctl get postgresql
postgresql_class=daemon
postgresql_flags=NO
postgresql_logger=
postgresql_rtable=0
postgresql_timeout=30
postgresql_user=_postgresql

puffy# rcctl getdef postgresql
postgresql_class=daemon
postgresql_flags=-D /var/postgresql/data -w -l /var/postgresql/logfile
postgresql_logger=
postgresql_rtable=0
postgresql_timeout=30
postgresql_user=_postgresql

```
<br/>
<br/>

The `get` subcommand reports no flags, but the `getdef` (for *get defaults*) reports the current settings of the daemon. Clearly, what we are interested in is the `postgresql_flags`. There are two ways to make a change to a value of the `rcctl` variables:
- editing the rc script by hand;
- using `rcctl` to set the value.

The latest is the preferred way, but, hey, this is Unix, so you can also fire up your favourite editor and go change the `/etc/rc.d/postgresql` file, that in fact appears as:

<br/>
<br/>
```shell
puffy# less /etc/rc.d/postgresql

#!/bin/ksh
#
# $OpenBSD: postgresql.rc,v 1.13 2019/08/27 19:49:46 awolk Exp $

daemon="/usr/local/bin/pg_ctl"
daemon_flags="-D /var/postgresql/data -w -l /var/postgresql/logfile"
daemon_user="_postgresql"

. /etc/rc.d/rc.subr

...
```
<br/>
<br/>

Clearly, editing this file by hand can be error prone and must be done with the cluster (i.e., PostgreSQL) not running, or you can result in not being able to stop the instance (e.g., changing the `PGDATA`).
<br/>

Using `rcctl` can do something better than manually editing the script file, but it has some constraints:
- the daemon must be system wide enabled;
- only *rc variables* can be edited (i.e., you cannot define your own variables);
- a variable is named without the daemon prefix.

<br/>
Therefore, in order to change both the `PGDATA` and the logging directory and file, we can edit the `flags` variable as follows:

<br/>
<br/>
```shell
puffy# rcctl enable postgresql

puffy# rcctl set postgresql flags "-D /var/postgresql/13/data -l /var/postgresql/13/log/postgresql.log" 

puffy# rcctl get postgresql
postgresql_class=daemon
postgresql_flags=-D /var/postgresql/13/data -l /var/postgresql/13/data/log/postgresql.log
postgresql_logger=
postgresql_rtable=0
postgresql_timeout=30
postgresql_user=_postgresql

```
<br/>
<br/>

Of course, you have to create the new `PGDATA` and the logging directory by hand, assign the right ownership to `_postgresql` before you can start the service.

<br/>
Please also note that if you disable, at system-wide level, the daemon, the customized configuration will be lost:


<br/>
<br/>
```shell
puffy# rcctl get postgresql
postgresql_class=daemon
postgresql_flags=-D /var/postgresql/13/data -l /var/postgresql/13/data/log/postgresql.log
postgresql_logger=
postgresql_rtable=0
postgresql_timeout=30
postgresql_user=_postgresql

puffy# rcctl disable postgresql

puffy# rcctl get postgresql     
postgresql_class=daemon
postgresql_flags=NO
postgresql_logger=
postgresql_rtable=0
postgresql_timeout=30
postgresql_user=_postgresql

```
<br/>
<br/>



## Make PostgreSQL start at boot

In order to let OpenBSD start PostgreSQL at boot, you have to enable the service system-wide. This can be achieved, as already shown, by means of the `enable` command, or by setting the `stauts` variable to `on`:


<br/>
<br/>
```shell
puffy# rcctl enable postgresql

# the same
puffy# rcctl set postgresql status on

```
<br/>
<br/>

## The *At-Boot* Configuration

Once the service is enabled at system-wide level, it can be customized by means of the `rcctl set` command, as already shown. The reason is that, once a daemon is *enabled* at boot, its name is appended to the list of serices in the file `/etc/rc.conf.local`, that is in turn used to determine what to start at boot.
<br/>
The custom configuration goes in that file too, and once the daemon is disabled, the configuration is scrubbed out of the file, so that only the default values (in the rc-script) survive:


<br/>
<br/>
```shell
puffy# rcctl enable postgresql
puffy# rcctl set postgresql flags "-D /var/postgresql/13/data -l /var/postgresql/13/data/log/postgresql.log" 

puffy# cat /etc/rc.conf.local
amd_flags=
pkg_scripts=postgresql transmission_daemon
postgresql_flags=-D /var/postgresql/13/data -l /var/postgresql/13/data/log/postgresql.log
```
<br/>
<br/>

In the above, there are two services that have been installed on the system: PostgreSQL and Transmission. The system is going to start PostgreSQL first (because it is leftmost), and then Transmission. When starting PostgreSQL, it is going to use the specified flags.
<br/>
If the PostgreSQL is now disabled, the setting are also lost:

<br/>
<br/>
```shell
puffy# rcctl disable postgresql
puffy# cat /etc/rc.conf.local   
amd_flags=
pkg_scripts=transmission_daemon
```
<br/>
<br/>

## Decide When to Start at Boot

It is also possible to let PostgreSQL start after (or before) specific other daemons. If we re-enable PostgreSQL, it will be appended into the `rc.conf.local` file, and therefore it will be started after the Transmission daemon; this can be obtained also from the `rcctl order` command:


<br/>
<br/>
```shell
puffy# rcctl enable postgresql
puffy# rcctl order             
transmission_daemon postgresql

```
<br/>
<br/>

Let's say we want PostgreSQL to be started as soon as possible, it is possible to change the order of starting by means of `rcctl order` command: you need to specify the leftmost (absolute first) daemon to start, or the list of daemons you want to start in the beginning:


<br/>
<br/>
```shell
puffy# rcctl order postgresql
puffy# rcctl order            
postgresql transmission_daemon

```
<br/>
<br/>


## Two is Better Than One

What if you want another PostgreSQL instance controlled by `rcctl`?
<br/>
You can copy the rc-script, giving another name, and chagne the set of flags to let it start:

<br/>
<br/>
```shell
puffy# cp /etc/rc.d/postgresql /etc/rc.d/postgresql_replica                                                                                       
puffy# rcctl enable postgresql_replica
puffy# rcctl set postgresql_replica flags "-D /var/postgresql/replica/data -l /var/postgresql/replica/data/log/postgresql.log -o '-p 5433'"
puffy# mkdir -p /var/postgresql/replica/data
puffy# chown -R _postgresql:_postgresql /var/postgresql/replica/data

puffy# su - _postgresql
puffy$ initdb /var/postgresql/replica/data
...
puffy$ mkdir /var/postgresql/replica/data/log

puffy# rcctl start postgresql_replica
postgresql_replica(ok_
```
<br/>
<br/>

There is some work to perform, but it is quite simple after all.


## So, is PostgreSQL Running?

Besides checking for allowed connections, you can use `rcctl` to see if the daemon is running: the `ls` command accepts a *status* you are looking for, `started` for running daemons, and returns the running services:


<br/>
<br/>
```shell
puffy# rcctl ls started
...
postgresql
postgresql_replica
...

```
<br/>
<br/>


# Conclusions

PostgreSQL can, of course, run well on OpenBSD systems. It can also be managed via the integrated service handler, named `rcctl` and its *rc-scripts*, as well as manually by means of PostgreSQL utility (e.g., `pg_ctl`).
