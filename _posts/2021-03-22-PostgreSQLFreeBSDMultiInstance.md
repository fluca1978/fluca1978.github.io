---
layout: post
title:  "Managing Multiple PostgreSQL Instances on FreeBSD"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- freebsd
permalink: /:year/:month/:day/:title.html
---
FreeBSD `service(8)` is a fully featured system to manage services, and allows multiple instances of PostgreSQL.


# Managing Multiple PostgreSQL Instances on FreeBSD

FreeBSD allows the management of multiple instances of PostgreSQL by means of `rc.conf(5)`.
<br/>
The trick is to use **profiles**, that are available for the PostgreSQL rc script (`/usr/local/etc/rc.d/postgresql`) even if not well documented, at least in my opinion.
<br/>
In order to understand how to deal with multiple PostgreSQL instances, consider a system with two cluster: *test* and *prod*.
<br/>
In `/etc/rc.conf` you need to define the `postgresql_profiles` variable, where you list the clusters separated by spaces. Then, for each profile, you define the well know `postgresql_xxx` variables, specifying the profile name before the variable suffix. For example, to define a `PGDATA`, that will be usually defined into `postgresql_data` variable, you need to specify a `postgresql_<profile-name>_data` variable.
<br/>
Therefore, in `/etc/rc.conf` you need to specify the following:

<br/>
<br/>
```shell
postgresql_profiles="test prod"

postgresql_test_data="/postgres/12/test"
postgresql_test_enable="YES"

postgresql_prod_data="/postgres/12/prod"
postgresql_prod_enable="YES"
```
<br/>
<br/>

Now you need to manage all instances by specifying the profile name on every `service(8)` call:

<br/>
<br/>
```shell
% sudo service postgresql start test

% sudo service postgresql status test
pg_ctl: server is running (PID: 35979)
/usr/local/bin/postgres "-D" "/postgres/12/test"
```
<br/>
<br/>

**You need to specify the profile name as last argument to `service(8)` invocation**.
<br/>
But there is more: **if you don't specify any profile on the command line, `service(8)` will iterate on all available profiles**. As an example, the following two sequences are equivalent:

<br/>
<br/>
```shell
% sudo service postgresql stop       
===> postgresql profile: test
===> postgresql profile: prod

# equivalent to
% sudo service postgresql stop test
% sudo service postgresql stop prod
```
<br/>
<br/>

With this simple profile-based management, it is easy to handle and manage multiple PostgreSQL instances on the same FreeBSD host.

