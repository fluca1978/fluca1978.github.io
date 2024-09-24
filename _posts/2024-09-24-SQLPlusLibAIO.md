---
layout: post
title:  "Oracle SQLPlus and libaio not found on Ubuntu"
author: Luca Ferrari
tags:
- oracle
- ubuntu
- linux
permalink: /:year/:month/:day/:title.html
---
A noisy problem due to the renaming of a library...

# Oracle SQLPlus and libaio not found on Ubuntu

I tend to use `sqlplus` as a way to connect to Oracle, since I'm used to the great `psql` command line client for PostgreSQL.
I mean, I use `sqlplus` despite I know how poor it is with regard to `psql`, but this is not the story this post is about.

One day, after the usual *break-everything* update of Ubuntu, my `sqlplus` stopped working:

<br/>
<br/>
```shell
% sqlplus ...
sqlplus: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
```
<br/>
<br/>

Therefore, I checked the `libaio` status:

<br/>
<br/>
```shell
% aptitude search libaio
i   libaio-dev                                     - Linux kernel AIO access library - development files
i   libaio-dev:i386                                - Linux kernel AIO access library - development files i   libaio1t64                                     - Linux kernel AIO access library - shared library    i   libaio1t64:i386                                - Linux kernel AIO access library - shared library
```
<br/>
<br/>

So everything was in place, but please note that the `libaio1` required by `sqlplus` has been renamed to `libaio1t64`!

And `sqlplus` is not able to deal with it, clearly!

<br/>
<br/>
```shell
% ldd /opt/oracle/instantclient_21_11/sqlplus
        linux-vdso.so.1 (0x00007ffdaf6af000)
        libsqlplus.so => /opt/oracle/instantclient_21_11/libsqlplus.so (0x0000773756e00000)
        libclntsh.so.21.1 => /opt/oracle/instantclient_21_11/libclntsh.so.21.1 (0x0000773752800000)
        libclntshcore.so.21.1 => /opt/oracle/instantclient_21_11/libclntshcore.so.21.1 (0x0000773752000000)
        libnnz21.so => /opt/oracle/instantclient_21_11/libnnz21.so (0x0000773751600000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x0000773757334000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x0000773757249000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x0000773757244000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x000077375723f000)
        libaio.so.1 => not found
        libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x000077375722c000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x0000773751200000)
        /lib64/ld-linux-x86-64.so.2 (0x0000773757358000)
        libaio.so.1 => not found
        libaio.so.1 => not found
        libaio.so.1 => not found

```
<br/>
<br/>



The trivial solution **was to link the `libaio1` object file to a new name, so that `sqlplus` can "find" it**:

<br/>
<br/>
```shell
% sudo ln -s /lib/x86_64-linux-gnu/libaio.so.1t64.0.2 /lib/x86_64-linux-gnu/libaio.so.1


% ldd $(which sqlplus)
        linux-vdso.so.1 (0x00007ffc659f4000)
        libsqlplus.so => /opt/oracle/instantclient_21_11/libsqlplus.so (0x00007c5526000000)
        libclntsh.so.21.1 => /opt/oracle/instantclient_21_11/libclntsh.so.21.1 (0x00007c5521a00000)
        libclntshcore.so.21.1 => /opt/oracle/instantclient_21_11/libclntshcore.so.21.1 (0x00007c5521200000)
        libnnz21.so => /opt/oracle/instantclient_21_11/libnnz21.so (0x00007c5520800000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007c55264ee000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007c5526403000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007c55263fe000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007c55263f9000)
        libaio.so.1 => /lib/x86_64-linux-gnu/libaio.so.1 (0x00007c55263f4000)
        libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007c55263e1000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007c5520400000)
        /lib64/ld-linux-x86-64.so.2 (0x00007c5526512000)

```
<br/>
<br/>


**ATTENTION**: so far `sqlplus` seems to work regularly, but I'm not sure this is an official way of fixing the problem, so do it at your own risk!
