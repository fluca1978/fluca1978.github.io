---
layout: post
title:  "PgBouncer gets SCRAM!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org


permalink: /:year/:month/:day/:title.html
---
A few of days ago a new release of PgBouncer has been released, with the addition of SCRAM support!

# PgBouncer gets SCRAM!
Three days ago [PgBouncer 1.11](https://pgbouncer.github.io/changelog.html#pgbouncer-111x) has been released, and one feature that immediately caught my attention was the addition of /SCRAM support for password/.
<br/>
<br/>
[`SCRAM`](https://www.postgresql.org/docs/11/auth-password.html) is currently the most secure way to use password for PostgreSQL authentication and has been around since version ~10~ (so nearly two years). `SCRAM` support for PgBouncer has been a /wanted feature/ for a while, since not having it prevented users of this great tool to use `SCRAM` on the clusters.
<br/>
<br/>
Luckily, now this has been implemented and [the configuration of the PgBouncer account](https://pgbouncer.github.io/config.html#authentication-file-format** is similar to the plain and ~md5~, so it is very simple.
<br/>
<br/>
I really love PgBouncer and, with this addition, I can now upgrade my servers to /SCRAM/!
**Thank you PgBouncer developers!**
