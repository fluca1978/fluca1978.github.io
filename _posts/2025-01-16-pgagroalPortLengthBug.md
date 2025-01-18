---
layout: post
title:  "The importance of testing with not-so-usual setups"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
How we discovered a trivial bug in pgagroal

# The importance of testing with not-so-usual setups

This week we found a trivial and silly bug in `[pgagroal](https://github.com/agroal/pgagroal){:target="_blank"}`.

This post is a brief description about such bug, not because it is important on itself, but because the way we discovered it emphasizes how important it is to *randomize* the configuration of a system.
It is a well known concept, however we all still tend to fail on this, due also to the lack and time to configure and test all possibilities (thanks God there is automation!).

As it often happens, the bug was caused by a memory allocation problem.

As it often happens in these cases. the fixing is a very short troophy patch.

Again, the aim of this post is not to discuss a *one line patch*, rather the importance of running and testing with different tools and setups.

## The memory bug

The bug [is described in a dedicated issue](https://github.com/agroal/pgagroal/issues/491){:target="_blank"}. What is interesting, as often happens when dealing with bugs, is how long it get unnoted.

While working and testing other *work in progress* features of `pgagroal`, I was encouraged to compile the project using `clang` instead of my usual `gcc`. The result was discouraging, since I was not able anymore to start the program:

<br/>
<br/>
```shell
% pgagroal
pgagroal: Unknown key <ev_backend> with value <io_uring> in section [pgagroal] (line 46 of file </etc/pgagroal/pgagroal.conf>)
2025-01-13 12:38:09 WARN  configuration.c:482 pgagroal: max_connections (20) is greater than allowed (8)
2025-01-13 12:38:09 DEBUG configuration.c:3074 PID file automatically set to: [/tmp/pgagroal.54322.pid]
=================================================================
==17659==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x502000009495 at pc 0x559726597f10 bp 0x7ffc9c1e2360 sp 0x7ffc9c1e1b00
WRITE of size 6 at 0x502000009495 thread T0
    #0 0x559726597f0f in vsprintf (/usr/local/bin/pgagroal+0x58f0f) (BuildId: 16ffc1dab018cfa8eed6b5cc7e6981bc6e861325)
    #1 0x55972659900e in sprintf (/usr/local/bin/pgagroal+0x5a00e) (BuildId: 16ffc1dab018cfa8eed6b5cc7e6981bc6e861325)
    #2 0x7f1cd144820b in bind_host /home/luca/pgagroal/src/libpgagroal/network.c:613:4
    #3 0x7f1cd1447bfc in pgagroal_bind /home/luca/pgagroal/src/libpgagroal/network.c:104:17
    #4 0x55972664e91c in main /home/luca/pgagroal/src/main.c:961:11
    #5 0x7f1cd10295cf in __libc_start_call_main (/lib64/libc.so.6+0x295cf) (BuildId: d78a44ae94f1d320342e0ff6c2315b2b589063f8)
    #6 0x7f1cd102967f in __libc_start_main@GLIBC_2.2.5 (/lib64/libc.so.6+0x2967f) (BuildId: d78a44ae94f1d320342e0ff6c2315b2b589063f8)
    #7 0x559726571a94 in _start (/usr/local/bin/pgagroal+0x32a94) (BuildId: 16ffc1dab018cfa8eed6b5cc7e6981bc6e861325)

0x502000009495 is located 0 bytes after 5-byte region [0x502000009490,0x502000009495)
allocated by thread T0 here:
    #0 0x55972660d04d in calloc (/usr/local/bin/pgagroal+0xce04d) (BuildId: 16ffc1dab018cfa8eed6b5cc7e6981bc6e861325)
    #1 0x7f1cd14481a3 in bind_host /home/luca/pgagroal/src/libpgagroal/network.c:607:12
    #2 0x7f1cd1447bfc in pgagroal_bind /home/luca/pgagroal/src/libpgagroal/network.c:104:17
    #3 0x55972664e91c in main /home/luca/pgagroal/src/main.c:961:11
    #4 0x7f1cd10295cf in __libc_start_call_main (/lib64/libc.so.6+0x295cf) (BuildId: d78a44ae94f1d320342e0ff6c2315b2b589063f8)

```
<br/>
<br/>

If I wasn't so lazy to test, even occasionally, another compiler, I would have discovered the problem sooner.

**Lesson learned #1**: using a different toolchain can speed up the discover of issues.

However, *we were testing and building `pgagroal` on different environments and by different toolchains*, hence how did this get unnoted?

Simple answer: **because developers tend to be lazy**. We tend to use the same setup over and over, and to follow the guides and howtos.


## Understanding the bug

The stacktrace reports a problem about a `calloc` call and something about a `5 byte region':

<br/>
<br/>
```shell
...
0x502000009495 is located 0 bytes after 5-byte region [0x502000009490,0x502000009495)
allocated by thread T0 here:
    #0 0x55972660d04d in calloc (/usr/local/bin/pgagroal+0xce04d) (BuildId: 16ffc1dab018cfa8eed6b5cc7e6981bc6e861325)
...
```
<br/>
<br/>

and luckily enough, we get also a line number and a file to look at:

<br/>
<br/>
```shell
...
    #2 0x7f1cd144820b in bind_host /home/luca/pgagroal/src/libpgagroal/network.c:613:4
...
```
<br/>
<br/>


Let's start from there. The code within `network.c`, around that line, was doing the following:

<br/>
<br/>
```c
char* sport;

sport = calloc(1, 5);
```
<br/>
<br/>


At its gist, the code is allocating a string to handle a number that represents a TCP/IP port number. The usage of `calloc` simplifies the well known pattern `malloc` plus `memset` to zero fill the memory.

It is quite simple now to spot the bug: a TCP/IP port upper boundary is `65535`, five digits, but the string needs the `\0` terminator.
And this is why all of this was unnoted before: the guide for `pgagroal` suggests to use the TCP/IP port `2345` (the reverse of the PostgreSQL default port) to listen for connections. Since `2345` is made by four digits, there is room for the string terminator.

However, on my setup, I use the port `54322`, which is five digits, hence the string terminator overflows the `sport` calloc-ated buffer.

The troophy patch was embarassing ([see this commit](https://github.com/fluca1978/pgagroal/commit/aca795b0bd8aaa137f9137f4b3ccf8f79c6bc00c){:target="_blank"}):

<br/>
<br/>
```
-   sport = calloc(1, 5);
+   sport = calloc(1, 6);
```
<br/>
<br/>

**Lesson learned #2**: use a not standard setup in order to look for problems.


# Conclusions

This short story emphasizes, once again, how important it is to change your own development environment and toolchain, as well as setup, in order to ease and speed the identification of problems.
If I did not change the toolchain, I wouldn't have seen the problem. And if I was not using a different setup than the "default" one, I wouldn't have seen the problem.

**Note that both the conditions had to happen for we to discovered the problem.**

And this is the important remark about the whole story.
