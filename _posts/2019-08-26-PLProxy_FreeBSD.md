---
layout: post
title:  "PL/Proxy on PostgreSQL 11 and FreeBSD 12"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- freebsd

permalink: /:year/:month/:day/:title.html
---
PL/Proxy is a procedural language implementation that makes really easy to do database proxying, and sharding as a consequence. Unluckily getting it to run on PostgreSQL 11 and FreeBSD 12 is not for free.

# PL/Proxy on PostgreSQL 11 and FreeBSD 12

[PL/Proxy](https://plproxy.github.io/) is a project that allows database proxying, that is a way to connect to remote databases, and as a consequence allows for /sharding/ implementations.
<br/>
The idea behind PL/Proxy is as simple as elegant: define a minimalistic language to access remote (database) objects and, more in particular, execute queries.
<br/>
<br/>
Unluckily, the latest stable release of PL/Proxy is `2.8` and is dated *October 2017*, that means *PostgreSQL 10*! There are a couple of Pull Requests to make it working against PostgreSQL 11, but hey have not been merged and the project code seems in pause.
<br/>
<br/>
Today I created a [cumulative pull request](https://github.com/plproxy/plproxy/pull/37) that does a little adjustments to allow the compilation on FreeBSD 12 against PostgreSQL 11.
<br/>
<br/>
My pull request is inspired and borrows changes from other two pull requests:
- [pr-31](https://github.com/plproxy/plproxy/pull/31) and credits to [Laurenz Albe](https://github.com/laurenz);
- [pr-33](https://github.com/plproxy/plproxy/pull/33) that has been merged into mine, and credits to [Christoph Berg](https://github.com/df7cb).
<br/>
Then I added a compiler flag to adjust headers on FreeBSD 12, as well as dropped an old Bison syntax since this should be safe enough on modern PostgreSQL (at least 9.6 and higher.
Some bit here and there to make all tests to pass against PostgreSQL 11, and everything seems right now.
<br/>
**It is important to warn that [my version](https://github.com/plproxy/plproxy/pull/37) is not *production ready* because it should be reviewed by at least one PL/Proxy developer**.

## And what about PostgreSQL 12?

Well, PostgreSQL 12 drops the usage of the special column `Oid` in catalogs, with commit 578b229718e8f15fa779e20f086c4b6bb3776106. What this means is that the macro `HeapTupleGetOid` is no longer there and PL/Proxy does an heavy usage of it. [I've tried to blindly substitute it with `->t_tableOid`](https://github.com/fluca1978/plproxy/tree/pg12), but this does not seems to work since the tests are failing to lookup objects. So any suggestion here is welcome!
