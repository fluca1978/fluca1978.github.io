---
layout: post
title:  "Sqitch and the special chars in the password (and not only there)"
author: Luca Ferrari
tags:
- sqitch
- oracle
permalink: /:year/:month/:day/:title.html
---
How to properly deal with passwords that contain special chars.

# Sqitch and the special chars in the password (and not only there)

One of my wills for the 2025 is to use the great [Sqitch](https://sqitch.org){:target="_blank"} database management tool as much as possible.
So, while being PostgreSQL my database of choice, I have to deal with several Oracle instances, so why not manage their changes thru `sqitch`?

So far, so good: I've already adopted this strategy for an appropriate set of databases. However today I encountered a problem, which lead me to an embarissing question posted on the [sqitch forum](https://github.com/sqitchers/sqitch/discussions/871){:target="_blank"}, that I solved by myself after a little more digging into the documentation.

The problem was about dealing with an `uri` where the password contained a `?` character, but the problem could arise with pretty much every other special char.

Well, after having looked at the online documentation for some help, the solution was proposed by the trivial integrate `help` function about `target`:


<br/>
<br/>
```shell
% sqitch help target

...
"uri"
        The database connection URI for the target. Required. Its format is:

          db:engine:[dbname]
          db:engine:[//[user[:password]@][host][:port]/][dbname][?params][#fragment]

        Some examples:

        "db:sqlite:widgets.db"
        "db:pg://dba@example.net/blanket"
        "db:mysql://db.example.com/"
        "db:firebird://localhost//tmp/test.fdb"

        Note that, as with any URI or URL, special characters must be URL
        encoded <https://en.wikipedia.org/wiki/URL_encoding>.
...
```
<br/>
<br/>


The last sentence is, obviously, what I'm interested in: **special chars have to be encoded!**


*Hence, substituting in the `uri` the `?` with **`3F`** (its encoded vesion) fixed the problem.*


It is, however, interesting to note that `DBI` does not handle the same mechanism: using an encoded single `uri` to `connect` does not work, while handling the `uri, usename, password` tuple works fine without any need for special care.
