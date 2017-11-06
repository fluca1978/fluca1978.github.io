---
layout: post
title:  "xmin and ON_ERROR_ROLLBACK"
author: Luca Ferrari
tags:
- PostgreSQL
permalink: /:year/:month/:day/:title.html
---
I hit what I thought it was a strange MVCC behavior, and of course PostgreSQL was right, just I was using the wrong option...

## xmin and ON_ERROR_ROLLBACK

Today I was querying a table on a database and tried to see if I remember right on MVCC, i.e., *Multi Version Concurrency Control*.
However I did noted what I thought it was a strange behavior: within a `BEGIN TRANSACTION` block the `xmin` was *always* increasing.

That puzzled me, so I decided to reproduce it on a short example and [asked for some explaination on the `-general` mailing list](https://www.postgresql.org/message-id/flat/CAKoxK%2B5Wm6RazPnU8AqB97XRqx5zbY7us00QSVrQgobbgmf8hQ%40mail.gmail.com#CAKoxK+5Wm6RazPnU8AqB97XRqx5zbY7us00QSVrQgobbgmf8hQ@mail.gmail.com).
First try was that there was something like a *trigger* (or an *event trigger*) using some sort of *sub-transaction*, but I did not have any. However this made me think about the trial of `logical decoding` I tried a few weeks ago on the very same cluster, so I asked if that was the problem.

But the answer was really easier and the behavior was due to the configuration of `psql`: I did not remember I had set on startup the `ON_ERROR_ROLLBACK` option. Despite its name, the option does the opposite: a transaction is already forced to rollback once a command issued an error.
The `psql` documentation reveals the problem:

```
 ON_ERROR_ROLLBACK
   When set to on, if a statement in a transaction block generates an error, the error is ignored and the transaction continues.
   When set to interactive, such errors are only ignored in interactive sessions, and not when reading script files. When unset or
   set to off, a statement in a transaction block that generates an error aborts the entire transaction. The error rollback mode
   works by issuing an implicit SAVEPOINT for you, just before each command that is in a transaction block, and then rolling back
   to the savepoint if the command fails.
```

in other words, having it to `on` makes `psql` to issue a `SAVEPOINT` before any command in the transaction block, therefore making the transaction itself made up by several sub-transactions, and that was the reason why `xmin` was increasing.

**PostgreSQL is, of course, always right!**
