---
layout: post
title:  "pgenv gets patching support"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
`pgenv` does now support a customizable *patching* feature that allows the user to define which patches to apply when an instance is built.


# pgenv gets patching support
[`pgenv`](https://github.com/theory/pgenv), the useful tool for managing several PostgreSQL installations, gets support for customizable patching. 

What is all about?
Well, it happens that you could need to patch PostgreSQL source tree before you build, and it could be because something on your operating system is different than the majority of the systems PostgreSQL is built against. Nevermind, you need to patch it!

`pgenv` did support a very simple patching mechanism hardcoded within the program itself, but during the last days I worked on a different and more customizable approach. The idea is simple: the program will apply every patch file listed in an *index* for the particular version. So, if you want to build the outshining 11.0 and need to patch it, build an index text file and list there all the patches, and the `pgenv` build process will apply them before compiling.

Of course, what if you need to apply the same patches over and over to different versions? You will end up with several indexes, one for each version you need to patch. Uhm...not so smart! To avoid this, I designed the patching index selection in a way that allows you to group patches for operating system and brand.

Allow me to explain more in detail with an example.
Suppose you are on a Linux machine and need to patch version 11.0: the program will search for a file that matches any of the following:

```
$PGENV_ROOT/patch/index/patch.11.0.Linux
$PGENV_ROOT/patch/index/patch.11.0
$PGENV_ROOT/patch/index/patch.11.Linux
$PGENV_ROOT/patch/index/patch.11
```

This *desperate* searching for works selecting the *first* file that matches the operating system and PostgreSQL version or a combination of the two including the major (or brand in previous versions) number.

Last, but not least, a new configuration variable has been introduced: `PGENV_PATCH_INDEX`. The usage of this variable allows you to overide the index selection mechanism providing a list of patches to apply that have a name possibly unlrelated at all with PostgreSQL version and/or operating system. Therefore, this allows you to do something like:

```
% PGENV_PATCH_INDEX=patch/patch_for_osx.txt pgenv build 11.0
```


I really hope this suffices in covering enough use cases to make this greate tool more and more useful.

None of the two above versions have been selected as default version, in order to use a version the `use` command must be issued:

```shell
% pgenv use 9.6.10
The files belonging to this database system will be owned by user "luca".
This user must also own the server process.
...
Logs are in [/home/luca/tmp/pgsql/data/server.log]
```

The very first time a database is selected its `PGDATA` directory is properly initialized by `initdb`.`
The `PGDATA` directory is set to `pgsql/data`, while the cluster has been linked to `pgsql` directory:

```shell
% ls -l pgsql
lrwxr-xr-x  1 luca  luca  12 Aug 30 06:59 pgsql -> pgsql-9.6.10
```

If I inspect again the installed versions, an asterisk will mark the currently selected version as the one in use:

```shell
% pgenv versions
       10.5    pgsql-10.5
 *   9.6.10    pgsql-9.6.10

% pgenv version 
9.6.10
```

The `versions` command (mind the *s*) shows all installed versions, while `version` shows the currently selected one.

### Start and Stop

The `start` and `stop` commands can be used to activate and deactivate the currently selected cluster:

```shell
% pgenv start
PostgreSQL 9.6.10 is already running

% pgenv stop 
waiting for server to shut down.... done
server stopped
PostgreSQL 9.6.10 stopped
```

### Clear and Nuke
The `clear` command unsets the currently selected version, that is *un-use* a version:

```shell
% pgenv clear
PostgreSQL 9.6.10 cleared

% pgenv versions
       10.5    pgsql-10.5
     9.6.10    pgsql-9.6.10
```

The `remove` command nukes the specified version, removing all the files and database content.

```shell
% pgenv remove 9.6.10
PostgreSQL 9.6.10 removed

% pgenv versions     
       10.5    pgsql-10.5
```

## There is more
There are other commands and usage tricks, please read the [documentation](https://github.com/theory/pgenv/blob/master/README.md) for a deeper understanding of this tool.

