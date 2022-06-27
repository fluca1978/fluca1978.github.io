---
layout: post
title:  "A new pgenv release"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv
permalink: /:year/:month/:day/:title.html
---
A new release of the PostgreSQL virtual environment manager.

# A new `pgenv` release

Today we [released version 1.3.1 of pgenv](https://github.com/theory/pgenv/releases/tag/v1.3.1){:target="_blank"}, the [binary manager for PostgreSQL](https://github.com/theory/pgenv){:target="_blank"}.
<br/>
This release fixes an annoying bug introduced on Mac OSX (Bash) that was preventing `pgenv` to properly work on such platform. The bug was introduced with [a commit of mines](https://github.com/theory/pgenv/commit/a547e3eaa4d21da5838fc6aeb0326f2c66a7b604){:target="_blank"} that was fixing another bug about wrong configuration reload.
<br/>
Unluckily, such bug gets unnoted because I don't have access to a Mac OSX, and I was quite sure Bash was much more portable than what we discovered!
<br/>
<br/>
Anyway, sorry for the bug, and please update your version of `pgenv` and enjoy it!
