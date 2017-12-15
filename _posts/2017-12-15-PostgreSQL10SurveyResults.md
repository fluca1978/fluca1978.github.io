---
layout: post
title:  "The most exciting feature of PostgreSQL 10 was..."
author: Luca Ferrari
tags:
- postgresql
permalink: /:year/:month/:day/:title.html
---
There was a survey, well there still is such a survey about the most exciting feature of PostgreSQL 10. Results are not what I was expecting for.

# The most exciting feature of PostgreSQL 10 was...

There is a survey on the [PostgreSQL](https://www.postgresql.org) main site that titles as **[What PostgreSQL 10 Feature are you most excited about?](https://www.postgresql.org/community/survey/93-what-postgresql-10-feature-are-you-most-excited-about/).

Now quick, what could be the results?

Surprisingly, at least for me, the most exciting feature is **native table partitioning**. Why is that surprising? For a couple of reasons: first of all PostgreSQL 10 is the first edition that ships with *logical replication* in core, and second table partitioning has been a supported feature for quite a long time, apparently every release I ever used in production did allowed me to partition a table.

How do I interpret such results? Well, I believe that simply people is satisfied by the binary/streaming replication so far, and while it is important to get also logical replication in the field, *more expressive tools are needed* by users. And this of course means native table partitioning.

This is somehow in contrast with a [much more old survey about 9.3](https://www.postgresql.org/community/survey/85-whats-your-favorite-93-feature/) where the trend seemed to be ahead low level features (memory, page checksums, etc.).
