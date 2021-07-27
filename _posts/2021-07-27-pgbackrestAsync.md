---
layout: post
title:  "pgbackrest async behavior" 
author: Luca Ferrari
tags:
- postgresql
- pgbackrest
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---

pgbackrest can work in asynchronous way in order to improve the resource usage.

# pgbackrest async behavior

[pgbackrest](https://pgbackrest.org/configuration.html){:target="_blank"} is an **amazing backup tool**, it is rock-solid (as PostgreSQL is) and designed to work under heavy database load.
<br/>
One feature it has to improve efficienty of WAL archiving is the *async* mode.
<br/>
<br/>
In "standard" mode, `pgbackrest` will *push* WAL segments to the backup machine, using the *classical* `archive_command` provided by PostgreSQL. As you probably already know, PostgreSQL will wait for `archive_command` to complete and acknowledge the WAL transfert. It could happen that:
- the `archive_command` could take a very long time, and while PostgreSQL will continue to work, not yet transferred WALs will make `pg_wal` to grow;
- the `archive_command` could fail, and PostgreSQL will warn you (in the logs) about this event and will try again to archive the failed WALs (forever, or better, unless it succeed).

<br/>
On the other hand, when doing a restore, PostgreSQL executes the `restore_command` to get a new WAL segment, and this in turn results in running `pgbackrest` for a single WAL request.
<br/>
The key concept here is probably **single WAL request**, both for push and get.
<br/>
<br/>
`pgbackrest` allows for an improvement on this situation by means of *asynchronous archive management*, both `push` and `get`. The idea is to give more control to `pgbackrest` so that it can optimize I/O operations.
<br/>
When PostgreSQL archives a WAL segment, it executes the `archive_command` within a loop (allow me to simplify things): when a WAL is ready, `archive_command` is invoked and until it has finished, there is no chance to archive an already available WAL segment. On the other hand, when PostgreSQL needs to get a WAL in order to do a restore/recovery, it executes `restore_command` on every WAL segment it is expecting to replay. Therefore, if the server has to replay many WALs, it has to execute `restore_command` and "download" every WAL one after the other.
<br/>
How does the asynchronous mode improve on the above?
<br/>
When archiving, that means *pushing*, `pgbackrest` can decide to group several WALs in a single transfert, that means for instance to reduce the setup/tear-down operations for establishing a network connection with the backup machine.
<br/>
When restoring, that means *getting*, `pgbackrest` could perform a pre-fetch, downloading a few WALs on the local machine and make them available immediatly to the PostgreSQL server when needed.

## The test environment

In this post, I will demonstrate the usage of `pgbackrest` asynchronously using my usual two-machine setup:
- `miguel` is the PostgreSQL server, running Fedora Linux with PostgreSQL 13.3;
- `carmensita` is the backup machine, running Fedora Linux.

<br/>
`pgbackrest` is at version `2.34`.

## Asynchronous Configuration Parameters

There are a bunch of configuration parameters that can be configured within the `pgbackrest.conf` file or specified on the command line, as usual.
<br/>
The settings mainly regard the spool directory, the queues and the enabling of the asynchronous mode.

### Enabling or disabling the asynchronous mode
There is a single configuration parameter to enable the asynchronous mode: `async`.
By default this is false, meaning `pgbackrest` will work "normally" as you expect. Turning it on, will automatically make any `archive-get` and `archive-push` in asynchronous mode.

### The spool directory

In order to manage the async operations, `pgbackrest` creates on the PostgreSQL machine a *spool directory*, usually `/var/spool/pgbackrest` where it places an `archive` directory and a directory named after the server, or better, the *stanza*. Such directory could then be split into `in` or `out` for respectively `archive-get` and `archive-push`.
<br/>
The spool directory root can be defined with the `spool-path` configuration parameter.
<br/>
For example, given the stanza named `miguel`, the spool directory will either be `/var/spool/pgbackrest/archive/miguel/out` or `/var/spool/pgbackrest/archive/miguel/in`.
<br/>
<br/>
In the `out` directory the system will write book-keeping stuff, mainly small text files that will be used to identify at which point the archiving has arrived.
<br/>
In the `in` directory, the system will store incoming WALs ready to be restored from the PostgreSQL server.

### Queues

There are two different setting to manage the queues of `pgbackrest`:
- `archive-push-max-queue`;
- `archive-get-max-queue`.

<br/>
They configure the max size of the data enqueued for the push and get operations. When the queue is full, `pgbackrest` will behave differently depending on the operation that is ongoing, as explained below.


# Configuration

The backup machine, named `carmensita` has a `7etc/pgbackrest.conf` file configured as follows:

<br/>
<br/>
```shell
$ cat /etc/pgbackrest.conf
[global]
start-fast = y
stop-auto  = y
repo1-path = /backup/pgbackrest

repo1-retention-full=2

repo1-host-user = backup
log-level-console = info


[miguel]
pg1-host = miguel
pg1-path = /postgres/13/data

```
<br/>
<br/>
while on the PostgreSQL server machine, named `miguel` the `/etc/pgbackrest.conf` file is

<br/>
<br/>
```shell
[global]
repo1-path = /backup/pgbackrest
repo1-host-user = backup
log-level-console = info
repo1-host = carmensita



archive-async          = y
archive-push-queue-max = 500MB
spool-path             = /var/spool/pgbackrest
archive-get-queue-max  = 32MB

```
<br/>
<br/>

Last, the `archive_command` on the PostgreSQL machine is configured as follows:

<br/>
<br/>
```shell
archive_command = '/usr/bin/pgbackrest \
                    --pg1-path=/postgres/13/data \
                    --config=/etc/pgbackrest.conf \
                    --stanza=miguel \
                    archive-push %p'
archive_mode = on

```
<br/>
<br/>
Please note that the `archive-async` parameter is specified in the configuration, instead of setting it in the `archive-push` or `archive-get`. This simplifies, in my opinion, the usage of `pgbackrest`.

<br/>
<br/>
With all the above up and running, it is possible to see how the asynchronous mode works.

# Archiving (`archive-push`)

Let's start with the backup scenario, that is `archive-push`.

## When things go right

Let's see what happens when everything is fine: I launched a `pgbench` session in order to generate some traffic and, therefore, some WAL segment generation and archiving.
On one hand, `pgbench` was running as follows:

<br/>
<br/>
```shell
% pgbench -c 8 -T 120 -h miguel -U pgbench -n -P 5 pgbench
```
<br/>
<br/>

While `pgbench` is running, let's inspect what is happening on the PostgreSQL machine, with particular regard to the spooling folder:

<br/>
<br/>
```shell
# ls -1s /var/spool/pgbackrest/archive/miguel/out \
      && psql -h miguel -U postgres \
         -c 'select last_archived_wal from pg_stat_archiver;' postgres

0 000000070000014E00000015.ok

last_archived_wal     
--------------------------
 000000070000014E00000015


# # after a while

# ls -1s /var/spool/pgbackrest/archive/miguel/out \
        && psql -h miguel -U postgres \
           -c 'select last_archived_wal from pg_stat_archiver;' postgres

0 000000070000014E00000016.ok

last_archived_wal     
--------------------------
 000000070000014E00000016

```
<br/>
<br/>
As you can see, in the spool directory there will be an **empty file** named after the last archived WAL segment, that is the last segment sent to the backup machine, and the suffix `.ok`.
<br/>
In the PostgreSQL logs, there will be a notice when the `pgbackrest` completes the pushing (depending on the log level you configured):

<br/>
<br/>
```shell
INFO: pushed WAL file '000000070000014E000000AD' to the archive asynchronously
```



## When things go wrong

### First case: shutting down the backup machine

Assume the backup machine, `carmensita`, is turned off. The archiving cannot work, of course, and if you generate again some traffic on the PostgreSQL server (e.g., by using `pgbench` as shown above), the situation on the spool directory is:

<br/>
<br/>
```shell
# ls -1s /var/spool/pgbackrest/archive/miguel/out \
    && psql -h miguel -U postgres \
       -c 'select last_archived_wal, last_failed_wal from pg_stat_archiver;' postgres

4 global.error

    last_archived_wal     |     last_failed_wal      
--------------------------|--------------------------
 000000070000014E0000001A | 000000070000014E0000001B

```
<br/>
<br/>
The file `global.error` contains a textual description of what is happening:

<br/>
<br/>
```shell
# cat /var/spool/pgbackrest/archive/miguel/out/global.error 
103
unable to find a valid repository:
repo1: [UnknownError] remote-0 process on 'carmensita' terminated unexpectedly [255]: ssh: connect to host carmensita port 22: No route to hos
```
<br/>
<br/>

If you then restart the backup machine, so that the archiving starts working again, the situation on the spool directory is as follows:

<br/>
<br/>
```shell
# ls -1s /var/spool/pgbackrest/archive/miguel/out \
    && psql -h miguel -U postgres \
       -c 'select last_archived_wal, last_failed_wal from pg_stat_archiver;' postgres

0 000000070000014E0000001B.ok
0 000000070000014E0000001C.ok
0 000000070000014E0000001D.ok
0 000000070000014E0000001E.ok
0 000000070000014E0000001F.ok

    last_archived_wal     |     last_failed_wal      
--------------------------|--------------------------
 000000070000014E0000001F | 000000070000014E0000001B

```
<br/>
<br/>
As you can see, the `.ok` files are there and the archiving is working again.
<br/>
During the time, there could be one or more `.ok` files. The idea is that the *last* `.ok` file indicates the last asynchronously archived WAL segment (in the above, the one ending with `1F`).

## Second case: generating more WALs

Shutdown the backup machine again, so that the PostgreSQL server is not able to archive WAL segments; then generate quite an amount of traffic to increase the WAL directory size (`pg_wal`).
<br/>
Let's inspect the situation:

<br/>
<br/>
```shell
# ls -1s /var/spool/pgbackrest/archive/miguel/out \
     && psql -h miguel -U postgres \
        -c 'select last_archived_wal, last_failed_wal from pg_stat_archiver;' postgres

4 global.error

    last_archived_wal     |     last_failed_wal      
--------------------------|--------------------------
 000000070000014E000000ED | 000000070000014E000000EE
  
 # cat /var/spool/pgbackrest/archive/miguel/out/global.error 
103
unable to find a valid repository:
repo1: [UnknownError] remote-0 process on 'carmensita' terminated unexpectedly [255]: ssh: connect to host carmensita port 22: No route to host

```
<br/>
<br/>
Therefore, the `14E0xED` is the last archived WAL on the backup machine.
<br/>
Suppose now a larger amount of data is mangled on PostgreSQL, so that it starts generating WAL segments. Clearly PostgreSQL cannot archive segments anymore, and will start accumulating them into `pg_wal` to keep them available for when the `archive_command` will start to work again.
<br/>
Or does it?
<br/>
Inspect again the situation on disk:

<br/>
<br/>
```shell
# ls -1s /var/spool/pgbackrest/archive/miguel/out \
   && psql -h miguel -U postgres      \
     -c 'select last_archived_wal, last_failed_wal from pg_stat_archiver;' postgres

4 000000070000014F00000053.ok
4 000000070000014F00000054.ok
4 000000070000014F00000055.ok
4 000000070000014F00000056.ok
4 000000070000014F00000057.ok
4 000000070000014F00000058.ok
4 000000070000014F00000059.ok
4 000000070000014F0000005A.ok.pgbackrest.tmp

    last_archived_wal     |     last_failed_wal      
--------------------------|--------------------------
 000000070000014F00000059 | 000000070000014F00000053


# cat /var/spool/pgbackrest/archive/miguel/out/000000070000014F00000059.ok
0
dropped WAL file '000000070000014F00000059' because archive queue exceeded 500MB
  
```
<br/>
<br/>
First of all: **`last_archived_wal` advanced even if the `acrhive_command` is failing (remember that the backup machine is down)**! How is that possible?
<br/>
The answer is in how `pgbackrest` asynchronous works: **if the number of failed WALs is greater than a specified size, `pgbackrest` decides to ackwnloedge the archiving to the PostgreSQL server, that in turn advances in archiving even if _the archived WAL did not hit the backup machine!_**
<br/>
The idea is that `pgbackrest` will prevent the `pg_wal` to grow undefinetly, thus risking to stop PostgreSQL to work at all. However, ** acknowledging a *fake archiving* means that the WAL-stream is broken, so Point In Time Recovery will not be possible anymore around this "hole" and a new backup is _strongly recommended_!** 
<br/>
`pgbackrest` inserts an information into its `.ok` files, that now are non-empty and inform the administrator that the WAL segment has been dropped explicitly.
<br/>
You can find the same information into the PostgreSQL logs, where `pgbackrest` prints a message to make it clear:

<br/>
<br/>
```shell
% sudo grep 000000070000014F00000059 $PGLOG

INFO: archive-push command begin 2.34: [pg_wal/000000070000014F00000059] --archive-async --archive-push-queue-max=500MB --config=/etc/pgbackrest.conf --exec-id=40124-af251b1c --log-level-console=info --pg1-path=/postgres/13/data --repo1-host=carmensita --repo1-host-user=backup --repo1-path=/backup/pgbackrest --spool-path=/var/spool/pgbackrest --stanza=miguel

WARN: dropped WAL file '000000070000014F00000059' because archive queue exceeded 500MB

INFO: pushed WAL file '000000070000014F00000059' to the archive asynchronously

```
<br/>
<br/>
There are three pieces of information:
- `pgbackrest` tried to archive the WAL segment, failing;
- there is a `WARN` that informs you that `pgbackrest` instrumented PostgreSQL to drop the WAL file *as if it was archived correctly*;
- `pgbackrest` states that it has archived the file, so PostgreSQL can proceed to delete or recycle it.

<br/>
What and when does `pgbackrest` decides to give up and starts faking to PostgreSQL? The `acrhive-push-queue-max` configuration paramater establish how many data `pgbackrest` can fail behind the normal WAL operations before trying to make PostgreSQL delete segments.
<br/>
In my configuration, there is `archive-push-queue-max=500MB`, that means that after `500MB` of failed WALs, `pgbackrest` will start faking and there will be a hole into the WAL stream. Roughly, this corresponds to `32` failed WALs on a row.



## Parallel Processes

The configuration parameter `process-max` can be used to control how many *push workers* can be launched to serve the asynchronous system. Suppose that in the configuration there is `process-max = 4`, then during WAL archiving you could see something as follows in the process list:

<br/>
<br/>
```shell
# pstree -c  -A
systemd-|-NetworkManager-|-{NetworkManager}
       ...                                                                                        
        |-pgbackrest-|-pgbackrest---ssh
        |            |-pgbackrest---ssh
        |            |-pgbackrest---ssh
        |            |-pgbackrest---ssh
        |            `-ssh
       ...                                        
        |-postmaster-|-postmaster
        |            |-postmaster
        |            |-postmaster
        |            |-postmaster
        |            |-postmaster
        |            |-postmaster---pgbackrest
        |            |-postmaster
        |            `-postmaster
      ...
```
<br/>
<br/>
As you can see, PostgreSQL has launched `pgbackrest` (that is, is executing the `archive_command`), and there are four `pgbackrest` processes.
<br/>
If the system is pushing archives in synchronous mode, `process-max` is ignored.
<br/>
Every concurrent process will share an `exec-id` that identifies the batch to which the process belongs:


<br/>
<br/>
```shell
# pstree -A -c -a -l | grep pgbackrest
...
  |-pgbackrest --config=/etc/pgbackrest.conf --exec-id=46475-10e060a1
  |   |-pgbackrest --config=/etc/pgbackrest.conf --exec-id=46475-10e060a1
  |   |-pgbackrest --config=/etc/pgbackrest.conf --exec-id=46475-10e060a1
  |   |-pgbackrest --config=/etc/pgbackrest.conf --exec-id=46475-10e060a1
...
```
<br/>
<br/>


# Restoring (`archive-get`)

Let's do a restore from a recent backup:

<br/>
<br/>
```shell
% sudo systemctl stop postgresql-13.service
% sudo -u postgres pgbackrest --stanza miguel \
       --pg1-path /postgres/13/data --delta restore
       ...
       INFO: restore command end: completed successfully (69861ms)
```
<br/>
<br/>
During the restore, the `archive` directory within the spool directory of `pgbackrest` is cleaned, in particular the specific server directory `miguel` is removed, since no WAL archiving is in progress.
<br/>
The `postgresql.auto.conf` file contains the `archive-get` command ready to fetch the WAL segments:

<br/>
<br/>
```shell
% sudo cat /postgres/13/data/postgresql.auto.conf

# Recovery settings generated by pgBackRest restore on 2021-07-27 05:26:26
restore_command = 'pgbackrest --pg1-path=/postgres/13/data --stanza=miguel archive-get %f "%p"'

```
<br/>
<br/>

During the system startup, `pgbackrest` will get (as usual) WAL segments from the backup machine, but this time in an asynchronous way:


<br/>
<br/>
```shell
INFO: archive-get command begin 2.34: [000000070000014F000000A4, pg_wal/RECOVERYXLOG] --archive-async --archive-get-queue-max=32MB --exec-id=42831-f4ada646 --log-level-console=info --pg1-path=/postgres/13/data --repo1-host=carmensita --repo1-host-user=backup --repo1-path=/backup/pgbackrest --spool-path=/var/spool/pgbackrest --stanza=miguel

INFO: found 000000070000014F000000A4 in the archive asynchronously

INFO: archive-get command end: completed successfully (713ms)

```
<br/>
<br/>
The above is an excerpt of the PostgreSQL log.
In the meantime, the spool directory was populated with a `in` subdirectory for the server, and in such directory the incoming WALs were stored waiting to be replayed by the PostgreSQL server:

<br/>
<br/>
```shell
% sudo ls -1s /var/spool/pgbackrest/archive/miguel/in
16384 000000070000014F000000A4.pgbackrest.tmp

```
<br/>
<br/>
In this scenario, the `archive-get-queue-max` parameter can specify the size of *pre-fetched* WALs: `pgbackrest` will fetch and store in the spooling directory no more WAL segments than the specified amount. Unlike the push configuration, setting this parameter does not imply the system will throw away WALs.



# Conclusions

`pgbackrest` is an amazing backup tool, rock solid and with a lot of configuration parameters that can help improving the resource usage so that the backup and restore work fast and reliably even under heavy loads.
<br/>
The asynchronous mode can help improving performances by means of *batches* and *pre-fetching* of WAL segments. However, you need to be aware about the fact that, by design, asynchronous pushing of WALs could produce holes in the WAL stream if the archiving accumulates too much data.
<br/>
This is, in my opinion, an excellent feature, because in my experience I've seen many times a PostgreSQL server accumulating too much WAL segments (up to consuming all the storage) due to a faulty backup machine (or networking). After all, `pgbackrest` is ensuring you that a backup exists, and at least that your PostgreSQL server will not go read-only due to `archive_command` failing.
<br/>
<br/>
Love it or hate it!
