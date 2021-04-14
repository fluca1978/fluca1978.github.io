---
layout: post
title:  "pgBackRest 2.33 and multi repository"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-conf
- pgbackrest
permalink: /:year/:month/:day/:title.html
---
pgBackRest now supports multiple repositories! 


# pgBackRest 2.33 and multi repository

A few days ago a [new release of pgbackrest, the 2.33](https://github.com/pgbackrest/pgbackrest/releases/tag/release%2F2.33){:target="_blank"} has been released. This release improves a lot of things, in particular two of them caught my attention:
- multi repository support;
- custom configuration path.

<br/>
The former allows `pgbackrest` to perform a multiple backup scattared over different repositories, in other words it allows the backup to be *mirrored* across different storages.
<br/>
The second improvement fixes a few annoyances with non-Linux operating systems, such as FreeBSD.
<br/>
In the following I give a glance at both this improvements.


# Custom configuration path

FreeBSD and, most in general, non-Linux machines use different default configuration paths. For example, what is commonly used as `/etc` on Linux is usually `/usr/local/etc`. In previous releases, there was room for using the `--prefix` option during the `configure` phase, but this was tedious because there was the need to specify the path to non standard files manually before invoking the command.
<br/>
In other words:

<br/>
<br/>
```shell
archive_command = '/usr/local/bin/pgbackrest --pg1-path=/postgres/12/data \
                       --config=/usr/local/etc/pgbackrest.conf \
                       --stanza=miguel  archive-push %p'
archive_mode = on
```
<br/>
<br/>

The important part to note in the above snippet, is that on FreeBSD if you wanted to use the standard (from an operating system point of view) path for the configuration, `pgbackrest` did not have any clue about and would try to look up the configuration file as `/etc/pgbackrest.conf`. The solution was, of course, to specify the `--config` option with the appropriate file.
<br/>
<br/>
Things have changed in version 2.33, since the `configure` command now can instrument the `pgbackrest` binary to find out the correct configuration file:

<br/>
<br/>
```shell
% ./configure --help
 ...
 --with-configdir=DIR    default configuration path
 ...
``**
<br/>
<br/>


**The default configuration path remains `/etc/pgbackrest.conf`** but it is now possible to specify a default configuration file path at compile time, so that you don't have to repeat yourself with `--config` at every invocation.


# Multi Repository Support

This is a much more important improvement, at least in my opinion. `pgbackrest` has been designed with this feature in mind, but until now there was not support for multiple repositories.
<br/>
Thanks to multiple repositories you can now *scatter or even mirror* your backups across different storage systems, so for example you can have a local repository and a remote one (e.g., in one of the supported cloud storages), or you can *mount* different storages and have the backup to be mirrored across all of them.
<br/>
The advantage of this solution is that it **provides a better redundancy** in the case your *single-point-of-failure* backup storage dies.
<br/>
One thing to take into account when working with multiple repositories is that a few `pgbackrest` commands now require a repository specification other than the stanza. The rule of thumb is that whenever `pgbackrest` is able to find out which repository to use, it will do, and this applies to the case when a single repository is configured. In other words, backward compatibility is safe!
<br/>
<br/>
In the following, there will be two configured repositories on the same backup machine. While this is *a very bad idea*, because it emphasizes a single point of failure, it allows for a quick run on multiple repository setup. The `carmensita` machine will handle two different local repositories:
- `/backup/pgbackrest` is the main repository;
- `/backup/pgbackrest-mirror` is the secondary repository, attached to a different storage.


## In the beginning there was only `repo1`

With `pgbackrest` prior to version 2.33, you could not configure multiple repositories: the configuration did accept a `repo1` set of variables but it was unable to *handle* repositories with a specification different from 1. As an example, consider the following configuration:

<br/>
<br/>
```shell
[global]
start-fast = y
stop-auto  = y

repo1-path = /backup/pgbackrest
repo1-retention-full=2
repo1-retention-archive=5

repo2-path = /backup/pgbackrest-mirror
repo2-retention-full = 1
```
<br/>
<br/>

Such a configuration  produces an error even in version 2.32:

<br/>
<br/>
```shell
$ pgbackrest --stanza miguel stanza-create
ERROR: [032]: only repo1 may be configured
```
<br/>
<br/>


## Multiple Repositories

I have to confess that setting up `pgbackrest` for different repositories on the same machine was not as simple as I initially thought, but once again [thanks to very professional community behind this great product](https://github.com/pgbackrest/pgbackrest/issues/1361){:target="_blank"}  I was able to fix my setup:

<br/>
<br/>
```shell
[global]
start-fast = y
stop-auto  = y
repo1-path = /backup/pgbackrest

repo1-retention-full=2
repo1-retention-archive=5



repo2-path = /backup/pgbackrest-mirror
repo2-retention-full = 1

log-level-console = info


[miguel]
pg1-host = miguel
pg1-path = /postgres/12/data
pg1-host-user = postgres
```
<br/>
<br/>

while on the target machine the main configuration parameters are (`/usr/local/etc/pgbackrest.conf`):

<br/>
<br/>
```
[global]
repo1-path = /backup/pgbackrest
repo1-host-user = backup
repo1-host = carmensita


repo2-host = sheriff
repo2-host-user = backup
repo2-path = /backup/pgbackrest-mirror
```
<br/>
<br/>


### Creating a stanza

As you can imagine, the `stanza-create` command creates the stanza in all the repositories automatically:

<br/>
<br/>
```shell
$ pgbackrest --stanza miguel stanza-create
P00   INFO: stanza-create for stanza 'miguel' on repo1
P00   INFO: stanza-create for stanza 'miguel' on repo2
P00   INFO: stanza-create command end: completed successfully (1017ms)
```
<br/>
<br/>

## Executing a backup

It is now time to execute a backup and see what happens:

<br/>
<br/>
```shell
% pgbackrest --stanza miguel backup
...
INFO: repo option not specified, defaulting to repo1
...
INFO: new backup label = 20210413-105939F
INFO: backup command end: completed successfully (254377ms)
INFO: expire command begin 2.33: --exec-id=1606-12c0320b --log-level-console=info --repo1-path=/backup/pgbackrest --repo2-path=/backup/pgbackrest-mirror --repo1-retention-archive=5 --repo1-retention-full=2 --repo2-retention-full=1 --stanza=miguel
INFO: expire command end: completed successfully (59ms)
```
<br/>
<br/>

As you can see, since I did not specify any particular repository, the program **program automatically selects the first repository**.



## Mixed backups

Having a single repository active in the backup list means the backup status is *mixed*:

<br/>
<br/>
```shell
$ pgbackrest --stanza miguel info
stanza: miguel
    status: mixed
        repo1: ok
        repo2: error (no valid backups)
    cipher: none

    db (current)
        wal archive min/max (12): 0000000100000005000000F2/000000010000000600000004

        full backup: 20210413-105939F
            timestamp start/stop: 2021-04-13 10:59:39 / 2021-04-13 11:03:51
            wal start/stop: 000000010000000600000004 / 000000010000000600000004
            database size: 2.5GB, database backup size: 2.5GB
            repo1: backup set size: 142.8MB, backup size: 142.8MB
```
<br/>
<br/>

To some extent, the above is a *degraded* state, that means not all repositories are up with good backups.
<br/>
Note that the single backup info now has a final line that indicates the repository where the backup can be found.


## Specifying the repository for a backup

You can specify the `--repo` option to instrument `pgbackrest` on which repository to store the backup:

<br/>
<br/>
```shell
% pgbackrest --stanza miguel backup --repo 2
...
INFO: backup command end: completed successfully (4846ms)
```
<br/>
<br/>



## The situation on the repositories

The `info` command can, as always, display information about repositories and their content:

<br/>
<br/>
```shell
% pgbackrest --stanza miguel info
stanza: miguel
    status: ok
    cipher: none

    db (current)
        wal archive min/max (12): 0000000100000005000000F2/000000010000000600000016

        full backup: 20210413-105939F
            timestamp start/stop: 2021-04-13 10:59:39 / 2021-04-13 11:03:51
            wal start/stop: 000000010000000600000004 / 000000010000000600000004
            database size: 2.5GB, database backup size: 2.5GB
            repo1: backup set size: 142.8MB, backup size: 142.8MB

        full backup: 20210413-111525F
            timestamp start/stop: 2021-04-13 11:15:25 / 2021-04-13 11:19:37
            wal start/stop: 00000001000000060000000F / 00000001000000060000000F
            database size: 2.5GB, database backup size: 2.5GB
            repo2: backup set size: 142.8MB, backup size: 142.8MB

...
```
<br/>
<br/>

## One backup at a time

It is not possible, as far as I know, to instrument `pgbackrest` to do simultaneously backups on all the repositories. This means that **you are in charge of scheduling backups on all the repositories manually**!


## Archiving on all the repositories

The archiving, however, is done on all repositories at the same time.
However, as [explained here](https://github.com/pgbackrest/pgbackrest/issues/1361#issuecomment-817630417){:target="_blank"}, the `archive-push` will iterate on every repository to push the same WAL segment. What this mean is that, from a PostgreSQL perspective, if a repository fails to get the WAL (while the others succeed), PostgreSQL will think the archiving has failed and will retry later.
<br/>
One way to solve the problem is to use the `archive-push` asynchronous mode.




# Conclusions

I am very enthusiast about how `pgbackrest` is progressing and how it is enabling new features at every release.

