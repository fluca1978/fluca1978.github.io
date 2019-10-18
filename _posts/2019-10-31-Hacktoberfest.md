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


### `pgenv` various command implementations

As a request by an user, most notably a well known Perl user, I decided to implement two different commands in `pgenv`:
- `pgenv alias` [that allows to label PostgreSQL versions with mnemonic names](https://github.com/theory/pgenv/pull/36);
- `pgenv rebuild` [that allows to rebuild an already installed version](https://github.com/theory/pgenv/pull/37).

<br/>
Then, I decided to try to figure out some more enhancements to `pgenv`:
- `pgenv psql` [that allows you to run the right version of `psql`](https://github.com/theory/pgenv/pull/38) for the running PostgreSQL cluster;
- [scripts to be ran once an event happens](https://github.com/theory/pgenv/pull/39).


### ITPUG Website

**ITPUG cannot keep its website up-to-date**! 
<br/>
On October 3rd, PostgreSQL 12 was released. As of October 17th, the ITPUG web site did not mention the fact, so [I felt worth making them noticing the fact](https://github.com/ITPUG/www.itpug.org/pull/32).
<br/>
Yes, this is a kind of *troophy patch*, I know. But it is really ridcolous how ITPUG is behaving.
<br/>
In the case you didn't know, the [PostgreSQL European Conference](https://pgconf.eu) did took place in Milan this year, near the mid of October. I noticed that the ITPUG web-site was not mentioning it so [I opened an issue about that](https://github.com/ITPUG/www.itpug.org/issues/33).
<br/>
<br/>
Anyway, taking a look at the overall Pull Requests, you can see that, as of time of writing, there were [five of them still opened](https://github.com/ITPUG/www.itpug.org/pulls), one by me opened on December 2018! That's not the way to manage a project.

<center>
<img src="images/posts/hacktoberfest/2019/itpug.png" />
</center>
