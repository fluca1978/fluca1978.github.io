---
layout: post
title:  "yum upgrade postgresql11 panic!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
I have to say, I don't use CentOS very much and I'm not a good user of `systemd`, that is the reason why I got five minutes of pure fear!

# yum upgrade postgresql11 panic!

*How hard could it be to upgrade PostgreSQL within minor versions?*
<br/>
Usually it is very simple, and *it is very simple* but not when you don't know your tools!
<br/>
And in this case that's my fault.
<br/>
However, I'm writing this short note in order to avoid other people experience the same problem I had.

## The current setup

The machine is a CentOS 7 running PostgreSQL 11.1 installed by [packages provided by the PostgreSQL Global Development Group](https://yum.postgresql.org/).

## Preparing to upgrade

Of course, I took a full backup before proceeding, just in case. The cluster I'm talking about is a low traffice cluster with roughly ~12 GB~ of data, that is the backup and restore are not a *zero downtime* (and no, I'm not in the position of having a WAL based backup, but that's another story).
<br/>
Having a backup helps keeping the amount of panic at a fair level.

## Performing the upgrade

I do like `yum(8)` and its transactional approach.
Doing the upgrade was a matter of:

```shell
% sudo yum upgrade postgresql11
```

and all dependencies are, of course, calculated and applied. Then I confirmed, waited a couple of minutes for the upgrade to apply, and **I started keeping my breath**:

```shell
psql: could not connect to server: Connection refused
        Is the server running on host "xxx" (192.168.222.123) and accepting
        TCP/IP connections on port 5432?
```

## Inspecting and solving the problem

Apparently PostgreSQL has not been restarted after the upgrade, but what is worst **is that is not going to restart again**:

```shell
10:33:25 lnx168 systemd[1]: Starting PostgreSQL 11 database server...
10:33:25 lnx168 postgresql-11-check-db-dir[10214]: "/var/lib/pgsql/11/data/" is missing or empty.
10:33:25 lnx168 postgresql-11-check-db-dir[10214]: Use "/usr/pgsql-11/bin/postgresql-11-setup initdb" to initialize the database cluster.
10:33:25 lnx168 postgresql-11-check-db-dir[10214]: See /usr/share/doc/postgresql11-11.4/README.rpm-dist for more information.
10:33:25 lnx168 systemd[1]: postgresql-11.service: control process exited, code=exited status=1
10:33:25 lnx168 systemd[1]: Failed to start PostgreSQL 11 database server.
```

**What the hell!** (I'm allowed to spell it loud because my colleague was on vacation and I was alone in my office).
<br/>
First of all, **do not run `initdb` as suggested** because chances are you will destroy all your data. But that's a good hint about the problem: **systemd was trying to launch PostgreSQL with an empty PGDATA**.
<br/>
<br/>
Of course, the `PGDATA` was not empty and was still in place, but **`yum` upgraded my `systemd` configuration for PostgreSQL to the CentOS default**, therefore my file `/usr/lib/systemd/system/postgresql-11.service` was overriden without any advice!
<br/>
<br/>
And in fact, to confirm the above, I was able to start the server manually using `pg_ctl`, and at least I had the server running.
<br/>
<br/>
Now that the server is running, I have more time to inspect `/usr/lib/systemd/system/postgresql-11.service` and adjust the `PGDATA` parameter to the right value:

```shell
% sudo grep PGDATA /usr/lib/systemd/system/postgresql-11.service
Environment=PGDATA=/data/pgdata
```

I also double checked that the `systemd` startup script correctly links to the edited file:

```shell
$ ls -l /etc/systemd/system/multi-user.target.wants/postgresql-11.service
lrwxrwxrwx 1 root root 45 20 dic  2018 /etc/systemd/system/multi-user.target.wants/postgresql-11.service 
                                           -> /usr/lib/systemd/system/postgresql-11.service

```

Seems fine, right?

## Nested problems

No matter how fine the setup was, `systemd` still refused to restart the cluster:

```shell
$ sudo service postgresql-11 restart                      
Redirecting to /bin/systemctl restart postgresql-11.service
Job for postgresql-11.service failed because the control process exited with error code. See "systemctl status postgresql-11.service" and "journalctl -xe" for details.
```

For a reason I don't really know, it seems that `systemd` keeps track that it hasn't started the service, and that the latter is in failed mode. The solution was to manually stop the cluster via `pg_ctl` and that asks `systemd` to start it again, and this time it gets running.

# Fixing the problem with `systemd**: the right approach
<br/>
**updated on 2019-07-22**
<br/>

As pointed out by *Andrew Gierth* in a comment, editing the `systemd` unit service file is not the right approach to configure services. Here it is the right approach, so that my changes do not get overwritten by `systemd`:
1) run `systemctl edit postgresql-11`;
2) add a line with `Environment=PGDATA=/data/pgdata` within the `Service` section:
```shell
[Service]
Environment=PGDATA=/data/pgdata
```
3) inspect the service with `systemctl status postgresql-11`, that will show the following:

```shell
$ systemctl status postgresql-11
● postgresql-11.service - PostgreSQL 11 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-11.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/postgresql-11.service.d
           └─override.conf
   Active: active (running) since lun 2019-07-22 15:43:50 CEST; 31s ago
     Docs: https://www.postgresql.org/docs/11/static/
 Main PID: 16114 (postmaster)
   CGroup: /system.slice/postgresql-11.service
           ├─16114 /usr/pgsql-11/bin/postmaster -D /postgres/data
           ├─16116 postgres: logger   
           ├─16118 postgres: checkpointer   
           ├─16119 postgres: background writer   
           ├─16120 postgres: walwriter   
           ├─16121 postgres: autovacuum launcher   
           ├─16122 postgres: stats collector   
           ├─16123 postgres: pg_cron scheduler   
           └─16124 postgres: logical replication launcher   
```

The important part in the above is the **Drop-In** line that points to a freshly created directory `/etc/systemd/system/postgresql-11.service.d` with a single file, `override.conf` that contains the new `PGDATA` definition. In other words, `systemd` keeps the service units under its own control, and you have to create an `override.conf` file to place other variable values.

# Conclusions

Not knowing your tools, `systemd` in this case, can lead to panic when they do not behave as you expect.
Unluckily, there are too many little details to know about every different system, and I wish `systemd` becomes a little less rude and at least warns the user that his files are going to be overriden.
<br/>
While the unit file states, in its beginning, to not modify the file, it is not clear what is the best approach to use to re-define variables (include or override file?).
