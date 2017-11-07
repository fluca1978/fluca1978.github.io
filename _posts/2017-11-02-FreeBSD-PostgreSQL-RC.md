---
layout: post
title:  "Multicorn PostgreSQL FDW on FreeBSD"
author: Luca Ferrari
tags:
- PostgreSQL
- FreeBSD
permalink: /:year/:month/:day/:title.html
---
Installing Multicorn Foreign Data Wrapper on FreeBSD is not always as easy as it seems...

## Multicorn PostgreSQL FDW on FreeBSD
-----
I had to fight against [Multicorn](http://multicorn.org/) Foreign Data Wrappers because installing them on FreeBSD is not ideal.
It is worth noting that the platform has not released a new stable version since February 2016, while the [commit logs](https://github.com/Kozea/Multicorn/commits/master) shows active development in order to be compatible with PostgreSQL 10.

So what are problems installing Multicorn on FreeBSD? It is simple: **Multicorn has not been made for FreeBSD and therefore the toolchain is not portable**.
In particular, in order to install it you have to:
1. use `gmake` instead of `make`;
2. change the she-bang line of `prefly-check.sh` which refers to *Bash*. Yes, really, *Bash*!
   Therefore change the first line from `#!/bin/bash` to `#!/usr/local/bin/bash`;
3. install package `python-distutils`, for instance `py27-python-distutils-extra-2.39`, which in turn installs `setuptools` that is required by `multicorn`.

With the above, Multicorn can be installed, but there's another trick to be aware of: the [FDW Users' Guide](http://multicorn.org/foreign-data-wrappers/) reports to use in pretty much all the examples the `wrapper_multicorn` as FDW, but that is wrong.
The correct FDW to use is `multicorn`, as can be seen by the code within [multicorn.sql](https://github.com/Kozea/Multicorn/blob/master/sql/multicorn.sql):

```sql
CREATE FOREIGN DATA WRAPPER multicorn
```
So when you create a server with `CREATE SERVER` do use `multicorn` as `FOREIGN DATA WRAPPER`.

With the above tricks I was able to run the file system FDW, as well as the Git one (which is pretty much un-documented).
Unluckily, the GoogleFDW is both undocumented and not working, due to the [need for a Google API and Custom Search that I reported to the module author](https://github.com/Kozea/Multicorn/issues/199) after a little "debugging" (or kind of).
I also asked to the ITPUG mailing list for someone proficient in Python for a Pull Request, maybe the ITPUG will be able to contribute!
