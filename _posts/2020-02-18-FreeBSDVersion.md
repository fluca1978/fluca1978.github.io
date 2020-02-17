---
layout: post
title:  "Knowing the FreeBSD you are running"
author: Luca Ferrari
tags:
- freebsd
permalink: /:year/:month/:day/:title.html
---
There are occasions when you could be confused at what system your are mantaining.

# Knowing the FreeBSD you are running

Knowing what operating system version you are running is very important, especially in order to report bugs and open issues.
<br/>
Sysadmins all around the world are used to old friend `uname(1)` that reports, with different level of details, the version of the kernel:

```shell
luca@miguel ~ % uname -a
FreeBSD miguel 12.1-RELEASE-p1 FreeBSD 12.1-RELEASE-p1 GENERIC  amd64
```

The important information there is the `12.1-RELEASE-p1` part.
<br/>
However, `uname(1)` looks at the running kernel, and does not know anything about any update you could have done on your system.
<br/>
<br/>
Entering `freebsd-version`.
<br/>
<br/>

`freebsd-version` is a shell script that does a lot of efforts in trying to understand what you are managing. It accepts three options:
- `-k` to inspect the installed kernel;
- `-u` to inspect the installed userland;
- `-r** to inspect the running kernel.
<br/>
**Wait a minute: is it possible that the running kernel does not match the installed kernel?**
<br/>
Yes, in particular after an upgrade (e.g., `freebsd-upgrade`) that has not done yeta  reboot.
<br/>
Here it is an example from the very same system:

```shell
luca@miguel ~ % freebsd-version -k
12.1-RELEASE-p2
luca@miguel ~ % freebsd-version -r
12.1-RELEASE-p1
luca@miguel ~ % freebsd-version -u
12.1-RELEASE-p2
```

As you can see, the installed kernel and userlands are `12.1-RELEASE-p2` but the system is *still* running `12.1-RELEASE-p1`, that leads us to conclude an updated has finished but the server has not reboot yet.

<br/>
But how does `freebsd-version` understands the installed kernel if it is not running?
<br/>
The secret is behind a small program, `what(1)`, that analyzes an executables and extracts a text string that indicates the version of the linked binary:

```shell
luca@miguel ~ % what /boot/kernel/kernel
/boot/kernel/kernel:
        FreeBSD 12.1-RELEASE-p2 GENERIC
luca@miguel ~ % what -qs /boot/kernel/kernel
FreeBSD 12.1-RELEASE-p2 GENERIC
```

while the running kernel can be inspect into the `sysctl` set, as in

```shell
luca@miguel ~ % sysctl kern.osrelease
kern.osrelease: 12.1-RELEASE-p1
```

And what about the userland? Well, there is nothing around the system that indicates the userland version, therefore the trick is that a new userland installs a new `freebsd-version` with an hardcoded value:

```shell
luca@miguel ~ % grep USERLAND_VERSION $(which freebsd-version)
USERLAND_VERSION="12.1-RELEASE-p2"
        echo $USERLAND_VERSION
```
