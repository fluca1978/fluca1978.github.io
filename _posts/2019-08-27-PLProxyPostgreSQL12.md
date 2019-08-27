---
layout: post
title:  "PL/Proxy on PostgreSQL 12 ?"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
I spent some more time on the PL/Proxy code base in order to make it compiling against upcoming PostgreSQL 12.

# PL/Proxy on PostgreSQL 12 ?

In my [yesterday blog post](https://fluca1978.github.io/2019/08/26/PLProxy_FreeBSD.html) I reported some stupid thougth about compiling PL/Proxy against PostgreSQL 12.
<br/>
I was too stupid to hit the removal of `HeapTupleGetOid` (as of [commit 578b229718e8f15fa779e20f086c4b6bb3776106 ](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commit;h=578b229718e8f15fa779e20f086c4b6bb3776106)), and after having read the commit comment with more accuracy, I found how to fix the code (*at least I hope so!*).
<br/>
<br/>
Essentially, wherever I found usage of `HeapTupleGetOid` I placed a preprocessor macro to extract the `Form_pg_` structure and use the normal column `oid` instead, something like:


```c
#if PG_VERSION_NUM < 12000 
  Oid namespaceId = HeapTupleGetOid(tup);
#else
  Form_pg_namespace form = (Form_pg_namespace) GETSTRUCT(tup);
  Oid       namespaceId  = form->oid;
#endif
```

<br/>
**I strongly advise to not use this in production, at least until someone of the PL/Proxy authors have a look at the code**! However the tests pass on PostgreSQL 12beta2 on Linux.
<br/>
<br/>
You can find the [pull request](https://github.com/plproxy/plproxy/pull/38) that also includes my previous pull request to make PL/Proxy work against PostgreSQL11 and FreeBSD.
<br/>
I hope it can help pushing a new release of this tool.
