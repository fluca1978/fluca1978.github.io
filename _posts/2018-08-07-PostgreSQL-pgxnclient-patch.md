---
layout: post
title:  "pgxnclient and beta version"
author: Luca Ferrari
tags:
- postgresql
- python
- itpug
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
`pgxnclient` is a wonderful `cpan` like tool for the [PGXN](http://pgxn.org) extension network. Unlickily, the client cannot handle PostgreSQL beta version, so I submitted a really small patch to fix the issue.

# pgxnclient and beta version

If you, like me, are addicted to terminal mode, you surely love a tool like `pgxnclient` that allows you to install extension into PostgreSQL from the command line, much like `cpan` (and friends) does for Perl.

A few days ago, I run into a problem: the `load** command cannot work against a PostgreSQL 11 beta 2 server. At first I reported it with a [ticket])https://github.com/dvarrazzo/pgxnclient/issues/29), but then curiosity hit me and I decided to give a look at very well written source code.

**Warning: I'm not a Python developer**, or better, I'm a Python-idiot! This means the work I've done, even if it seems it works, could be totally wrong, so reviews are welcome.

First I got to the regular expression used to parse a `version()` output:

```python
m = re.match(r'\S+\s+(\d+)\.(\d+)(?:\.(\d+))?', data)
```

where `data` is the output of a `SELECT version();`. Now, this works great for a version like `9.6.5` or `10.3`, but does not work for `11beta2`. Therefore, I decided to implement a two level regular expression check: at first search for a two or three numbers, and if it fails, search for two numbers separated by the `beta` text string.

```python
 m = re.match(r'\S+\s+(\d+)\.(\d+)(?:\.(\d+))?', data)
 if m is None:
     m = re.match( r'\S+\s+(\d+)beta(\d+)', data )
     is_beta = True
     if m is None:
         raise PgxnClientException(
             "cannot parse version number from '%s'" % data)
 else:
    is_beta = False
```

[Apparently it works](https://github.com/fluca1978/pgxnclient/commit/9ddce97679f3e2af6aaa3c8bb9ec90e62a3ffb87), but I'm not sure if there are not other pieces of code that need more attention.
