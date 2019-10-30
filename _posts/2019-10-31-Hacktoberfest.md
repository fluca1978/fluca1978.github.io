---
layout: post
title:  "Hacktoberfest 2019"
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

## `pg_proctab` on FreeBSD

While preparing material for a PostgreSQL professional course, I found [pg_proctab](https://github.com/markwkm/pg_proctab).
Since my default PostgreSQL machine runs on FreeBSD, and `pg_proctab` did not compile on such an operating system, I decided to try to make it work. So [here it is my attempt at making it compiling on FreeBSD](https://github.com/markwkm/pg_proctab/pull/5). After several days, I realized that probably the project on GitHub was not under the radar, and in fact it was just a mirro, so I pushed the *Merge Request* on[GitLab too](https://gitlab.com/pg_proctab/pg_proctab/merge_requests/1).


## `pgBackRest` command `expire` with `--dry-run`

This started as a very simple patch, at least I thought, and revelead itself as one of the most complex in this October, to the extent it is still open!
The idea is to provide a `--dry-run` option to the `expire` command to let an user to see what is going to happen on backups without removing them.
I have to say the team behind `pgBackrest` has been very supporting and polite, and I hope to be able to finish this [pull request](https://github.com/pgbackrest/pgbackrest/pull/853).

## `pgenv` improvements

As usual, I did some improvements to `pgenv`:
- [implement an `alias` command](https://github.com/theory/pgenv/pull/36) in order to let the user provide mnemonic names to each installed cluster. This is till ongoing work, since I later decided to try to implement a whole aliasing mechanism in order to allow for multiple installations;
- [implement a `rebuild` command](https://github.com/theory/pgenv/pull/37), a quite straightforward patch to implement a `rebuild` command that starts a total rebuild of the specified cluster;
- [implement a `psql` command](https://github.com/theory/pgenv/pull/38), something I then closed by myself to substitute with the actual implementation of a *warning* about the user's `PATH`. The idea was to provide the user with the actual PostgreSQL executables related to the in-use cluster;
- [supports scripts](https://github.com/theory/pgenv/pull/39) allowing the user to define custom scripts to be executed at different phases during the build/start-up of a cluster.

## `pgconf.eu` and `www.itpug.org`

Well, this is a kind of troophy patch that happened to be just because I simply don't understand the way ITPUG is handling the spread of PostgreSQL (and Open Source) in Italy.
In October PostgreSQL 12 was released, and the [official web site for the ITPUG community did not mention that](https://github.com/ITPUG/www.itpug.org/pull/32).

