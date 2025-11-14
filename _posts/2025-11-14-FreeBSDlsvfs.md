---
layout: post
title:  "FreeBSD lsvfs"
author: Luca Ferrari
tags:
- freebsd
permalink: /:year/:month/:day/:title.html
---
A nice command to get information about the supported filesystems.

# FreeBSD lsvfs

I discovered a FreeBSD specific command, **`lsvfs(1)`**, that is handy to get the list of the Virtual FileSystems allowed into the operating system.

The command is quite easy to read:

```shell
% lsvfs
Filesystem                              Num  Refs  Flags
-------------------------------- ---------- -----  ---------------
devfs                            0x00000071     1  synthetic, jail
msdosfs                          0x00000032     0
nfs                              0x0000003a     0  network
procfs                           0x00000002     0  synthetic, jail
tmpfs                            0x00000087     0  jail
cd9660                           0x000000bd     0  read-only
ufs                              0x00000035     0
zfs                              0x000000de    23  jail, delegated-administration
```

The name of the filesystem is the first column, while the magic number identifying the filesystem is the second one.
The `refs` column indicates how many instances of the filesystem are currently in use. As an example, the `zfs` row shows a counting `ref` of `23`, which is confirmed by a simple double check:

```shell
% zfs list -o canmount | grep on | wc -l
      23
```

that is there `23` *mounted* instances (out of `28` entries shown in `zfs list`).

The `flags` column shows some information about the filesystem, with the important evidence of **synthetic** that means that the filesystem is not a real storage, rather something that can be used via the I/O interface.
Other flags, like `jail` are quite obvious: the filesystem can be used within a jail; similarly `read-only` means, well, you cannot write to the filesystem,
It is interesting the `delegated-administration` flag, that is a quite fancy way to tell that the user can manage mounting.

Flags are detailed in the man page of `getvfsbyname(3)` as follows:

```
VFCF_STATIC      statically compiled into kernel
VFCF_NETWORK     may get data over the network
VFCF_READONLY    writes are not implemented
VFCF_SYNTHETIC   data does not represent real files
VFCF_LOOPBACK    aliases some other mounted FS
VFCF_UNICODE     stores file names as Unicode
VFCF_JAIL        can be mounted from within a jail if allow.mount and
                 allow.mount.<vfc_name> jail parameters are set
VFCF_DELEGADMIN  supports delegated administration if vfs.usermount
                 sysctl is set to 1
```


# Conclusions

FreeBSD is very consistent and has a lot of tiny commands that can help you understanding what is going on and how is configured the system.
