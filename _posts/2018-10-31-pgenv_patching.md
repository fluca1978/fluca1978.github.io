---
layout: post
title:  "pgenv gets patching support"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv
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

