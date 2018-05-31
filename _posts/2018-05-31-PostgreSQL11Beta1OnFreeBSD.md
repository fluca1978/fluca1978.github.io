---
layout: post
title:  "PostgreSQL 11 beta 1 on FreeBSD 11"
author: Luca Ferrari
tags:
- freebsd
- postgresql
permalink: /:year/:month/:day/:title.html
---
Compiling PostgreSQL 11 beta 1 on FreeBSD 11 is not hard, but special care has to be taken with regard to `readline`.

# Compiling PostgreSQL on FreeBSD: readline header not found

Long story short: when issueing a `configure` in your PostgreSQL source tree you got:

```sh
checking for readline.h... no
configure: error: readline header not found
```

and the `readline` library is installed on the system:

```sh
% pkg info readline       
readline-7.0.3_1
Name           : readline
Version        : 7.0.3_1
Installed on   : Thu May 31 13:39:23 2018 CEST
...
```

So what is going wrong? Simple: the `configure` script is not searching for in the correct include path, that on FreeBSD is `/usr/local/include`. The problem can be easily fixed adding `--with-includes` to the `configure` command line, and in particular the following is the line I use to compile PostgreSQL (e.g., 11 beta 1):

```sh
% ./configure --prefix=/opt/pg11b1 
                     --with-perl 
                     --with-python 
                     --with-openssl 
                     --with-libraries=/usr/local/lib 
                     --with-includes=/usr/local/include/   
```

The part `--with-includes=/usr/local/include/` fixes the error on readline.
