---
layout: post
title:  "Running pgbackrest on FreeBSD"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- freebsd
permalink: /:year/:month/:day/:title.html
---
pgbackrest is an amazing backup solution for PostgreSQL, quite frankly it is my favourite. And now fully supports FreeBSD too!

# Running pgbackrest on FreeBSD

[pgbackrest](https://pgbackrest.org/index.html){:target="_blank"} is an amazing tool for backup and recovery of a PostgreSQL database. 
Quite frankly, **it is my favourite backup solution** because it is reliable, fast and supports a lot of interesting features including retention policies and encryption.
<br/>
[I have already written](https://fluca1978.github.io/2019/03/04/pgbackrest_FreeBSD.html){:target="_blank"} about some problems in running [pgbackrest](https://pgbackrest.org/index.html){:target="_blank"} on FreeBSD, and the problem were not related to the application itself, rather to the compilation process.
<br/>
I'm really glad that now [pgbackrest](https://pgbackrest.org/index.html){:target="_blank"} fully supports non-Linux platforms, including FreeBSD, thanks to the changes in the compilation approach. It is therefore a simple process to get pgbackrest installed on your FreeBSD machine!



## Installing pgbackrest on FreeBSD

In order to see how simple it is now to install pgbackrest on FreeBSD, let's download the latest stable release, the `2.27` one, and install it. The only advice is that the project needs to be compiled with `GNU make`, that means you have to digit `gmake` inestead of usual `make`:

```shell
% wget https://github.com/pgbackrest/pgbackrest/archive/release/2.27.tar.gz
% tar xzvf 2.27.tar.gz
% cd pgbackrest-release-2.27

% cd src
% ./configure --prefix=/usr/local/pgbackrest
% gmake
% sudo gmake install
```

I've decided to install it on a specific path, `/usr/local/pgbackrest` just to avoid messing with other binaries, but you can install in the default FreeBSD location `/usr/local/`. If everything was succesful, you can then proceed to testing the program:


```shell
% export PATH=/usr/local/pgbackrest/bin:$PATH

% pgbackrest
pgBackRest 2.27 - General help

Usage:
    pgbackrest [options] [command]

Commands:
    archive-get     Get a WAL segment from the archive.
    archive-push    Push a WAL segment to the archive.
    backup          Backup a database cluster.
    check           Check the configuration.
    expire          Expire backups that exceed retention.
    help            Get help.
    info            Retrieve information about backups.
    restore         Restore a database cluster.
    stanza-create   Create the required stanza data.
    stanza-delete   Delete a stanza.
    stanza-upgrade  Upgrade a stanza.
    start           Allow pgBackRest processes to run.
    stop            Stop pgBackRest processes from running.
    version         Get version.

Use 'pgbackrest help [command]' for more information.
```

Great! Installing on FreeBSD is now really simple!

## Some recent history about pgbackrest

In the last few month the porject was deply improved, and I'm not going to quote the whole [release history](https://pgbackrest.org/release.html){:target="_blank"} here. However, there are two major aspects that I found really interesting.

### Migrating to C

[pgbackrest](https://pgbackrest.org/index.html){:target="_blank"} was initially developed mainly in Perl, with little parts written in C to deal with performances and internals of PostgreSQL WAL files format.
<br/>
As of January 2020, release `2.21`, the whole codebase is in C. Well, this is not fully true, since the testing and pre-configuration part is still written in Perl, at least to my understanding, but the whole `pgbackrest` production thing is now in C.
<br/>
The fact that the application is now written in C makes a clear distinction between `pgbackrest` and other similar backup solutions, that indeed take advantages of existing tools to behave as "glue" between small pieces. Moreover, it means that the backup, and most notably the restore, can run at full speed.


### My little messy contribution

A long time ago... I tried to contribute to a requested feature that sounded very easy to implement, and of course it was not!
<br/>
Since version `2.25` there is the `--dry-run` flag for the `expire` command:

> Add --dry-run option to the expire command. 
> Use dry-run to see which backups/archive would be removed by the expire command 
> without actually removing anything. (Contributed by Cynthia Shang, Luca Ferrari. 
> Reviewed by David Steele. Suggested by Marc Cousin.)

Unluckily, I was unable to complete the effort because I was unable to use the testing system, [and it was my fault](https://github.com/pgbackrest/pgbackrest/pull/853){:target="_blank"}, I underestimated the problem. But there are two very good news about this:
- the project provide me a very quick, polite and constant support in trying to fix my issues;
- they required me to test my changes instead of doing the testing by themselves.

Why are the above good news? First of all, other projcets are not so reactive when new contributions come, and I think this is very important for the project health. Second, testing a feature means that the project will not introduce regressions, and forcing every developer to test their own changes is a very good habit.


# Conclusions

I have already used `pgbackrest` on FreeBSD, but now that it is *natively** supporting this platform I believe that the project will attrac more and more users. Moreover, now that all the code has been converted to C, the already optimal performances will be much more impressive.
<br/>
<br/>
**[pgbackrest](https://pgbackrest.org/index.html){:target="_blank"} is definetely my backup solution of choice**, and not only for its features, but also for the clean and rigorous way the project is mantained and improved.
