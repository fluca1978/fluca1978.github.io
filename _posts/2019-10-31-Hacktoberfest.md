---
layout: post
title:  "Hacktoberfest 2018"
author: Luca Ferrari
tags:
- Hacktoberfest
- Open Source
permalink: /:year/:month/:day/:title.html
---
Another year, another october, another Hacktoberfest!

# Hacktoberfest 2019

## Glance at my Pull Request

Here is a short list about my Pull Requests and how they gone. The [overall status can be cheked as usual](https://hacktoberfestchecker.jenko.me/user/fluca1978).

### `pg_proctab` compiling on FreeBSD

While preparing material for a PostgreSQL professional course, I hit a problem with `[pg_proctab](https://github.com/markwkm/pg_proctab)`: the extension was not able to compile against FreeBSD 12.
<br/>
The problem was easy to fix, and required a single `#ifdef` statement to make [conditional compilation](https://github.com/markwkm/pg_proctab/pull/5).

### `pgbackrest expire --dry-run`

This is a very complex, at least to me, [pull request](https://github.com/pgbackrest/pgbackrest/pull/853) and tryies to implement a *test only* (dry-run) version of the `expire` command. It is based on initial work on [another pull request of mine](https://github.com/pgbackrest/pgbackrest/pull/840) that was the *connecting way* with the development team.
