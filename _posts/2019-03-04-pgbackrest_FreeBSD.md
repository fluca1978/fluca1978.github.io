---
layout: post
title:  "Running pgbackrest on FreeBSD"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- freebsd
permalink: /:year/:month/:day/:title.html
---
I tend to use FreeBSD as my PostgreSQL base machine, and that's not always as simple as it sounds to get software running on it. In this post I take some advices on running `pgbackrest` on FreeBSD 12.

# Running pgbackrest on FreeBSD

[pgbackrest](https://pgbackrest.org/index.html) is an amazing tool for backup and recovery of a PostgreSQL database. However, and this is not a critique at all, it has some Linux-isms that make it difficult to run on FreeBSD.
I tried to install and run it on FreeBSD 12, stopping immediatly at the compilation part. So [I opened an issue](https://github.com/pgbackrest/pgbackrest/issues/686) to get some help, and then tried to experiment a little more to see if at least I could compile.

The first trial was to cross-compile: I created the executable (`pgbackrest` has a single executable) on a Linux machine, then moved it to the FreeBSD machine along with all the `ldd` libraries (placed into `/compat/linux/lib64`). But `libpthread.so.0` prevented me to start the command:

```shell
% ./pgbackrest 
./pgbackrest: error while loading shared libraries: libpthread.so.0: 
  cannot open shared object file: No such file or directory
```

So I switched back to native compilation and, as described in the [issue](https://github.com/pgbackrest/pgbackrest/issues/686) I made a little changes to the `client.c` and the `Makefile`. Since it compiled (using of course `gmake`), I also made a little more changes to `Makefile` to compile and install it the FreeBSD way (i.e., under `/usr/local/bin`). The full diff is the following (some changes are not shown in the issue):

```shell
% git diff
diff --git a/src/Makefile b/src/Makefile
index 73672bff..0472c7f1 100644
--- a/src/Makefile
+++ b/src/Makefile
@@ -8,7 +8,7 @@
CC=gcc
# Compile using C99 and Posix 2001 standards (also _DARWIN_C_SOURCE for MacOS)
-CSTD = -std=c99 -D_POSIX_C_SOURCE=200112L -D_DARWIN_C_SOURCE
+CSTD = -std=c99 
# Compile optimizations
COPT = -O2
@@ -51,7 +51,7 @@ LDFLAGS = -lcrypto -lssl -lxml2 -lz $(LDPERL) $(LDEXTRA)
# Install options
####################################################################################################################################
# Modify destination install directory
-DESTDIR =
+DESTDIR = /usr/local/
####################################################################################################################################
# List of required source files.  main.c should always be listed last and the rest in alpha order.
@@ -175,8 +175,8 @@ pgbackrest: $(OBJS)
# Installation.  DESTDIR can be used to modify the install location.
####################################################################################################################################
install: pgbackrest
-       install -d $(DESTDIR)/usr/bin
-       install -m 755 pgbackrest $(DESTDIR)/usr/bin
+       install -d $(DESTDIR)bin
+       install -m 755 pgbackrest $(DESTDIR)/bin
####################################################################################################################################
# Compile rules
diff --git a/src/common/io/tls/client.c b/src/common/io/tls/client.c
index ddddb790..10b1d538 100644
--- a/src/common/io/tls/client.c
+++ b/src/common/io/tls/client.c
@@ -25,6 +25,7 @@ TLS Client
#include "common/type/keyValue.h"
#include "common/wait.h"
#include "crypto/crypto.h"
+#include <netinet/in.h>
/***********************************************************************************************************************************
Object type
```

Then, following the FreeBSD software paths, I created `/usr/local/etc/pgbackrest/pgbackrest.conf` and prooceed.
So far everything seems working, even if as far as I know, **FreeBSD is not a tested platform, so I'm working at my own risk (and so are you if you doing the same installation)!**

One little annoying detail is the configuration file: `pgbackrest` defaults to `/etc/pgbackrest/pgbackrest.conf`, and such file seems to me to be hardcoded into the `config/parse.c` source file:

```c
#define PGBACKREST_CONFIG_FILE                                      PROJECT_BIN ".conf"
#define PGBACKREST_CONFIG_ORIG_PATH_FILE                            "/etc/" PGBACKREST_CONFIG_FILE
STRING_STATIC(PGBACKREST_CONFIG_ORIG_PATH_FILE_STR,             PGBACKREST_CONFIG_ORIG_PATH_FILE);
```

or at least I don't see any comfortable way to change such behavior. The problem is that having to specify the FreeBSD-style configuration file `/usr/local/etc/pgbackrest/pgbackrest.conf` is not only annoying, but can cause weird errors, most notably an apparently unrelated error like

```shell
option pg1-path must be specified when relative wal paths are used
```

because the `archive_command` specified did not included the same configuration file and `pgbackrest` was looking for its default. In other words, ensures that the PostgreSQL instance has something like:

```shell
archive_command = '/usr/local/bin/pgbackrest 
                   --stanza=main 
                   --config=/usr/local/etc/pgbackrest/pgbackrest.conf  
                   archive-push %p'
```

That's made me think that linking `/usr/local/etc/pgbackrest` directory to `/etc/pgbackrest` could be, at this point a good solution to avoid some future mess.

