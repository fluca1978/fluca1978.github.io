---
layout: post
title:  "TRON Legacy: the console prompt"
author: Luca Ferrari
tags:
- Linux
permalink: /:year/:month/:day/:title.html
---
If you ever watched *TRON Legacy* you would have seen the console prompt and the commands on the terminal...

# TRON Legacy: the console prompt

A few evenings ago I was watching *TRON Legacy* with my son, and one short frame got my attention: the point when the Flynn's son finds the father laboratory and the (supposed) Unix machine. Days after, I watched again the same sequence to better understand the commands on the console.

![Console](https://i.redditmedia.com/evkGsrYrVVsXpobedPR7d0pH8UpjI3cSkIskdRrkpY0.jpg?w=1024&s=fc418226c579965a6a7ca406541f04a5)

First of all, the operating system type: the **SolarOS** (which reminds the *SunOS* and its layer *Solaris*) was supposed to be an operating system produced by the Flynn's *Encom* factory. What is quite strange is the architecture type: **sun4m i386** is a mix between a *Sparc* machine and an *Intel* one, that is something that *never existed*!

Apparently I was unable to find the `-n` option for `login`, so I don't know what a command like `login -n root` does, even if I can imagine it ask for logging as superuser. Moreover, I don't see why the login should be incorrect without prompting for a password.
After that, a `backdoor` login is tried (?) and the user results in a superuser (according to the `#` shell prompt); this is also confirmed for not having a *home directory*, and that reminds me some old versions of *SunOS* that placed the home directory of the root user into the slash.

The commands fromthe history appears even stranger:

![History](http://1.bp.blogspot.com/-SeqeMTrw7DI/UMYdl0u6rFI/AAAAAAAAADA/1x4meVdsnRI/s1600/whoami.png)

First of all, note that there is no executable named `history`, that is a shell level built-in (at least in every shell I know of).
Second, it should appear quite awkward that the command chain is:
1. `make`
2. `make install`
3. `configure`

with the last command obviously the first one in a build chain.

Even the line with the editing of the testament appears weird to me:

``` shell
vi ~/last_will_and_testament.txt
```

In fact, the special tilde `~` was a synonim for home directory only on recent shell, and not on a Bourne-like (as supposed to be in the old *SolarOS* computer back from 1985).

Moreover, I don't see the point in having a multitasking system to compile and build and *only after that* editing a text file, I would have done it in the meantime. Last, I don't see even the need for inspecting the available memory (`cat /proc/meminfo`), which reveals also a kind of *Linux-based kernel*.

Last but not least, all commands seem to be with a relative path and not the absolute one.

Nevertheless, I appreciate the effort in doing an almost coherent representation of a Unix system.
