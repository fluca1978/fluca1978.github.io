---
layout: post
title:  "Physical Backup Privileges Check"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A simple view to see if a user can perform backups.


# Physical Backup Privileges Check

In order to perform a physical backup, PostgreSQL requires a role that is allowed to perform several operations, mainly invoke `pg_start_backup()` and `pg_stop_backup()` functions.
<br/>
On old PostgreSQL versions, before version 10, only superusers can invoke the above backup functions and, therefore, can do a physical backup. Since PostgreSQL 10 things have changed and nowdays there are more fine grain permissions. In particular there are a few [default roles](https://www.postgresql.org/docs/12/default-roles.html){:target="_blank"} that can be used to set up a *backup role*.
<br/>
This is what I usually do, since working with superuser role can be dangerous, so I do create *low profile* roles and assign them the required privileges.
<br/>
<BR/>
Depending on the backup solution you are going to implement, these privileges can be different, so I decided to create a [view](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/backup_privileges.sql){:target="_blank"} that can help inspecting the status of the roles available in the database.
<br/>
<br/>
The view works as follows:
- extract a set of flags;
- merge with logical `AND` and `OR`;
- provide some additional flags.


<br/>
<br/>
Using the view results in something like the following:


<br/>
<br/>
```sql
backupdb=> select * from vw_role_backup_privileges
               WHERE rolname IN ( 'luca', 'backup' );
  -[ RECORD 1 ]------------|-------
  rolname                  | backup
  can_do_backup            | t
  can_monitor_backup       | t
  can_create_restore_point | t
  can_switch_wal           | t
  -[ RECORD 2 ]------------|-------
  rolname                  | luca
  can_do_backup            | f
  can_monitor_backup       | f
  can_create_restore_point | f
  can_switch_wal           | f
```
<br/>
<br/>
As you can see, the `backup` role, even if not a superuser, can do backups, can monitor them, can create restore points and force a WAL switch.
<br/>
Of course, the above is not a *one size fits all* solution, since every backup solution could require different permissions, however this is a possible starting point to check the status of the users.
<br/>
<br/>
In the following I describe the single pieces of the view.

## What is required to do a physical backup?

The minimal set of privileges required to perform a backup are:
- permission to start a replication;
- permission to invoke `pg_start_backup()`, `pg_stop_backup()`.

<br/>
This is done by the part:

<br/>
<br/>
```sql
f.can_start_replication
AND f.pg_start_backup
AND ( f.pg_stop_backup OR f.pg_stop_backup_exclusive )
```
<br/>
<br/>
where I check the above requirements.

## Additinal requirements

being able to start and stop a backup could not suffice: the user could be required to monitor the backup. Monitoring always means being able to query the statistic data and the configuration of the cluster. The former can be used to see if the replication is working fine, while the latter to check the archiving setup.
<br/>
PostgreSQL provides a `pg_monitor` role that can do the above queries, otherwise the user could need two different roles, namely `pg_read_all_settings` and `pg_read_all_stats`. Since `pg_monitor` includes the above two roles, assigning `pg_monitor` is equivalent to assign the latter two roles.
It could also be required to be able to query the `pg_is_in_backup()` function, that indicates if the cluster is actually in physical backup mode.
<br/>
This means that I need to check:

<br/>
<br/>
```sql
f.pg_monitor
  OR ( f.pg_read_all_settings
       AND f.pg_read_all_stats
       AND f.pg_is_in_backup
  )
```
<br/>
<br/>

## Switch WALs and create restore points

Starting a backup could also require the user to issue an immediate switch of the WALs in order to quickly start the backup. 
<br/>
Moreover, it could be required to create a restore point, for example to mark in the WALs that the backup has started at a specific point in time.
<br/>
This mean that the check is:

<br/>
<br/>
```sql
 f.pg_create_restore_point AS can_create_restore_point
 , f.pg_switch_wal           AS can_switch_wal
```
<br/>
<br/>


# Putting everything together

Having stated the above list of requirements, the query can be split into two parts:
- a CTE that extracts the flags;
- a query that composes the flags.
<br/>
<br/>
To extract the flags, the following CTE can be used:

<br/>
<br/>
```sql
WITH flags AS (
    SELECT
    a.rolname
    , a.rolsuper         AS is_superuser
    , a.rolreplication AS can_start_replication
    , pg_has_role( a.rolname, 'pg_monitor', 'USAGE' ) AS pg_monitor
    , pg_has_role( a.rolname, 'pg_read_all_settings', 'USAGE' ) as pg_read_all_settings
    , pg_has_role( a.rolname, 'pg_read_all_stats', 'USAGE' ) as pg_read_all_stats
    , has_function_privilege( a.rolname, 'pg_start_backup( text, bool, bool )', 'EXECUTE' ) as pg_start_backup
    , has_function_privilege( a.rolname, 'pg_stop_backup( bool, bool )', 'EXECUTE' ) as pg_stop_backup
    , has_function_privilege( a.rolname, 'pg_stop_backup()', 'EXECUTE' ) as pg_stop_backup_exclusive
    , has_function_privilege( a.rolname, 'pg_create_restore_point( text )', 'EXECUTE' ) as pg_create_restore_point
    , has_function_privilege( a.rolname, 'pg_is_in_backup()', 'EXECUTE' ) as pg_is_in_backup
    , has_function_privilege( a.rolname, 'pg_switch_wal()', 'EXECUTE' ) as pg_switch_wal
FROM
    -- use pg_roles instead of pg_authid
    -- to allow non-superuser roles to query
    pg_roles a
)
```
<br/>
<br/>

I do query `pg_roles` that contain all the information that is found in `pg_authid` but do not require superuser privileges to be queried.
<br/>
Please note that I check role group membership with the `USAGE` privilege, that means that the role does not have to do an explicit `SET ROLE` to gain access to the privileges from the group it belongs to, that is it has been created `WITH INHERIT`.
<br/>
<br/>
Then, composing the flags is as simple as:

<br/>
<br/>
```sql
SELECT f.rolname
    , f.is_superuser
      OR (
          f.can_start_replication
          AND f.pg_start_backup
          AND ( f.pg_stop_backup OR f.pg_stop_backup_exclusive )
      ) AS can_do_backup
    ,   f.pg_monitor
        OR ( f.pg_read_all_settings
             AND f.pg_read_all_stats
             AND f.pg_is_in_backup
        ) AS can_monitor_backup
    , f.pg_create_restore_point AS can_create_restore_point
    , f.pg_switch_wal           AS can_switch_wal
FROM flags f;
```
<br/>
<br/>

