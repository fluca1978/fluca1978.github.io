---
layout: post
title:  "Preventing FreeBSD to kill PostgreSQL (aka OOM Killer prevention)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- freebsd
permalink: /:year/:month/:day/:title.html
---
Something that can be useful when running PostgreSQL on FreeBSD.

# Preventing FreeBSD to kill PostgreSQL (aka OOM Killer prevention)

There are a lot of interesting articles on how to prevent the *Out of Memory Killer* (*OOM killer* in short) on Linux to ruin you day, or better your night. One particular well done explaination about how the OOM Killer works, and how to help PostgreSQL to survive, is, in my humble opinion, the one from [Percona Blog](https://www.percona.com/blog/2019/08/02/out-of-memory-killer-or-savior/){:target="_blank"}.

<br/>
<br/>
I tend to run PostgreSQL on FreeBSD machines, at least whenever it is possible, and quite frankly I have still a lot of things to learn. One of those *little* details is about _FreeBSD OOM Killer_.
<br/>
It turned out FreeBSD **has its own *OOM Killer implementation***, see [this excellent article](https://klarasystems.com/articles/exploring-swap-on-freebsd/){:target="_blank"}; I discovered it recently via the [excellent FreeBSD forum](https://forums.freebsd.org/threads/oom-killer-like-configuration.79514/){:target="_blank"} and, as usual, the kindness and professional of the community behind this great operating system.
<br/>
<br/>
A difference between Linux and FreeBSD is that the former exploits a lot the `/proc` filesystem to let the administrator to interact with the process configurations and information, while the latter does not.
And thanks to the [the above article](https://klarasystems.com/articles/exploring-swap-on-freebsd/){:target="_blank"} I discovered the `protect(1)` command, that is aimed to instrument the OOM Killer.
<br/>
<br/>
In the following I describe what I learnt so far and how to protect PostgreSQL from the OOM Killer.


## `protect(1)` and FreeBSD OOM Killer

Processes in FreeBSD has a particular flag named **`PROC_SPROTECT`** that, as the man page for `procctl(2)` system call states, is used to instrument the OOM Killer to skip this process when selecting a candidate to kill:

<br/>
<br/>
```
PROC_SPROTECT    Set process protection state.  This is used to mark
                 a process as protected from being killed if the
                 system exhausts the available memory and swap. 
```
<br/>
<br/>

The idea is that when the OOM Killer *scans* the processes to find out one (or more) candidate to kill to immediatly free memory, the *protected* processes must be skipped.
<br/>
An important thing to note is that **protection is not inherited by `fork(2)`-ed processes**. Luckily, it is possible to mark a protected process to let its children to inherit the protection status. In fact, setting `PROC_SPROTECT` to:
- `PPROT_SET` protects the current process but not its children;
- `PPROT_SET | PPROT_INHERIT` protects the current process and any children from hereby.

<br/>
Why is this detail important? Because as we all know, PostgreSQL starts with a *main* process (the *postmaster*) that forks a new process for every connection. Therefore, you are free to control the OOM Killer protection at level of postmaster or connection level.
<br/>
<br/>
*WARNING: marking all processes as protected can prevent the OOM Killer to work at all, with the presumably result of **panicing** the whole machine*.
<br/>
<br/>


## Protecting PostgreSQL from OOM Killer

There are two main ways to protect PostgreSQL from the OOM Killer:
- manually use `protect(1)` against one or more PostgreSQL processes;
- automatically use `protect(1)` at sevrice startup.

<br/>
Manually using `protect(1)` means that you are going to protect the process by means of its PID. As an example, suppose that on a machine there are the following processes:


<br/>
<br/>
```shell
% sudo pstree -s postgres
 \-+= 00776 postgres /usr/local/bin/postgres -D /postgres/12/data
   |--= 00777 postgres postgres: logger    (postgres)
   |--= 00779 postgres postgres: checkpointer    (postgres)
   |--= 00780 postgres postgres: background writer    (postgres)
   |--= 00781 postgres postgres: walwriter    (postgres)
   |--= 00782 postgres postgres: stats collector    (postgres)
   \--= 00783 postgres postgres: logical replication launcher    (postgres)
```
<br/>
<br/>

where the process with PID `776` is clearly the *postmaster*. Now, assume you want to protect the postmaster itself: you can call `protect(1)` specyfing the PID of the process.

<br/>
<br/>
```shell
% sudo protect -p 776
```
<br/>
<br/>

The main flags for `protect(1)` are:
- `-p` specifies the PID of the process to protect;
- `-d` or `-i` to apply the protection to all the current children or to the future children;
- `-c` to remove the protection.
<br/>
Therefore, in order to protect **all new connections to the database** the command to use is:

<br/>
<br/>
```shell
% sudo protect -i -p 776
```
<br/>
<br/>

that reads as *protect process `776` and all new forked processes*.
<br/>
<br/>
Doing all the protection manually is boring, and luckily the excellent `rc.d` system allows for the configuration of protection at the service startup. It is possible to specify the **`oomprotect`** configuration parameter for the service (all services, not only PostgreSQL!), that in turn can assume the following values:
- `yes` enables protection for (a single) process;
- `all` enables protection for all processes (forked).
<br/>
<br/>

**Unluckily, this does not apply directly to PostgreSQL since the `service(8)` script `/usr/local/etc/rc.d/postgresql` does not fully use `/etc/rc.subr` that, in turn, is in charge of examining the `oomprotect` variable.** The `postgresql` script uses directly `pg_ctl(1)` to manage the cluster, without any "protection** possible.
**I suspect the problem is due to the fact that `pg_ctl(1)` must be run as a normal user, and therefore there is the need to simultaneously run the `pg_ctl(1)` command without `root` privileges, as well as with such privileges to wrap it in `protect(1)`.**

In short, this means that even a configuration like the following will not apply `protect(1)` to PostgreSQL:



<br/>
<br/>
```shell
postgresql_enable="YES"
postgresql_data="/postgres/12/data"

# all = protect -i -p
# yes = protect -p
postgresql_oomprotect="all"
```
<br/>
<br/>


Therefore, in order to protect the *postmaster* or any other PostgreSQL process, you need to manually use `protect(1)` as already shown. 
<br/>
I am not sure if this is going to change in the future to allow the `rc.d` script to honor the `oomprotect` variable.

## How to inspect the protection status

This has been hard to me, but again thanks to great FreeBSD community and IRC, I discovered that `ps(1)` has a special command line argument, named `flags`, that can show the status of the single process protection. It is also the `flags2` command line argument that shows the status of the protection inheritance.
<br/>
Both the `flags` and `flags2` sections contain *hexadecimal values* that indicates all the extra information tied to a process. In the case of `P_PROTECTED` the value is `0x100000` (and this is found in `flags`), while for the `P_INHERIT_PROTECTED` the value is `0x00000001` (and this is found in `flags2`).
<br/>
Putting it all together, you can inspect your PostgreSQL processes as follows:

<br/>
<br/>
```shell
% sudo ps -ax -o flags,flags2,pid,command | grep postgres
10104000 00000001 3747 /usr/local/bin/postgres -D /postgres/12/data
10100000 00000001 3748 postgres: logger    (postgres)
10100000 00000001 3750 postgres: checkpointer    (postgres)
10100000 00000001 3751 postgres: background writer    (postgres)
10100000 00000001 3752 postgres: walwriter    (postgres)
10100000 00000001 3753 postgres: stats collector    (postgres)
10100000 00000001 3754 postgres: logical replication launcher    (postgres)
```
<br/>
<br/>

The first process, with PID `3747` is the already mentioned *postmaster* and it has a `flags` value of `10104000` that means **it is OOM protected**, and it also has a `flags2` section that is `00000001` that means it will make any spawn process protected too. 

You can check this with some math and Perl:

<br/>
<br/>
```shell

% sudo ps -ax -o flags,flags2,command \
       | grep postgres \  
       | perl -lanE 'say "[OOM PROTECTED]\t@F[2 .. $#F]" if $F[0] =~ /^\d{2}1\d{5}$/; '                |                                                                                |
[OOM PROTECTED] /usr/local/bin/postgres -D /postgres/12/data
[OOM PROTECTED] postgres: logger (postgres)
[OOM PROTECTED] postgres: checkpointer (postgres)
[OOM PROTECTED] postgres: background writer (postgres)
[OOM PROTECTED] postgres: walwriter (postgres)
[OOM PROTECTED] postgres: stats collector (postgres)
[OOM PROTECTED] postgres: logical replication launcher (postgres)
```
<br/>
<br/>

The above Perl one liner gets the command line and the `flags` section, as internal array `@F`, and checks if the third leftmost bit is set; in such case the process is protected against OOM killing.


## Hey 'ma, am I protected?

I created an example [pl/pgSQL function to check if the current connection is protected against the OOM Killer](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/freebsd_oom.sql){:target="_blank"}.
The function is defined with `SECURITY DEFINER` and has to be created by a superuser, because it internally uses the `COPY` command to execute the `ps` utility.

<br/>
<br/>
```sql
CREATE OR REPLACE FUNCTION
f_oomprotect( pid int DEFAULT NULL )
RETURNS boolean
AS
$CODE$
DECLARE
  p_protected  bit(8)  = '00100000';
  is_protected boolean = false;
  shell        text;
BEGIN
  -- if no pid supplied, use my own
  IF pid IS NULL OR pid < 0 THEN
    pid := pg_backend_pid();
  END IF;

  RAISE DEBUG 'Inspecting PostgreSQL process %', pid;

  shell :=    '/bin/ps -ax -o flags,flags2 -p '
                || pid
                || ' | /usr/bin/tail -n 1 ';
  CREATE TEMPORARY TABLE IF NOT EXISTS
            my_ps( flags bit(8), flags2 bit(8) );
  TRUNCATE my_ps;
  EXECUTE format( '  COPY my_ps( flags , flags2 ) FROM PROGRAM $$ %s $$ WITH ( DELIMITER $$ $$, FORMAT TEXT)', shell );


   SELECT ( flags & p_protected )::int > 0
   INTO is_protected
   FROM my_ps;

   RETURN is_protected;
END
$CODE$
LANGUAGE plpgsql
SECURITY DEFINER;
```
<br/>
<br/>

The idea is quite simple: the function gets a PID to check, if none is specified it assumes we are interested in the current connection. Then the function creates (or empties) a temporary table `my_ps` to store the result of the `ps` shell command, in particular `flags` and `flags2` (even if only the former is used).
Flags are stored as bit strings, so that it becomes simpler to make flag comparison.
Last, the `flags` field is compared with a logical and with the `p_protected` internal variable, and the boolean result is returned.
<br/>
Therefore if the function returns `true` the selected connection/backend process is protected against the OOM Killer.


# Conclusions

As usual FreeBSD reveals itself as a complex and well designed operating system. PostgreSQL can be protected against the OOM Killer in a more aggressive way with regard to Linux, but as usual *protecting everything is like protecting nothing at all*, so I recommend to not abuse about the `protec(1)` command.
