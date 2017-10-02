---
layout: post
title:  "GEOM Withering"
author: Luca Ferrari
tags:
- FreeBSD
- GEOM
permalink: /:year/:month/:day/:title.html
---
What does it mean that GEOMs can wither?

## GEOM Withering
-----

GEOM(8) is a great stack for managing storage medium, and it is in my opinion one of the great advantages of FreeBSD over other
operating systems. Nevertheless, GEOM can sometime act strangely with respect to what a sysadmin is expecting, and one
controversial feature is **wither**ing a provider.
But what is *withering*?

Imagine we have a disk, named **ada4** with a single *GPT* partition:

```sh
# glabel list ada4
Geom name: ada4
Providers:
1. Name: diskid/DISK-VB97f8d8f5-8f70a2a4
   Mediasize: 2147483648 (2.0G)
   Sectorsize: 512
   Mode: r0w0e0
   secoffset: 0
   offset: 0
   seclength: 4194304
   length: 2147483648
   index: 0
Consumers:
1. Name: ada4
   Mediasize: 2147483648 (2.0G)
   Sectorsize: 512
   Mode: r0w0e0
```

The disk is *unmounted*, so it can be find in the operating system as an inode in the ```/dev/``` filesystem:

```sh
# ls -l /dev/diskid/DISK-VB97f8d8f5-8f70a2a4 /dev/ada4p1
crw-r-----  1 root  operator  0x68 Oct  2 16:17 /dev/ada4p1
crw-r-----  1 root  operator  0x74 Oct  2 16:29 /dev/diskid/DISK-VB97f8d8f5-8f70a2a4
```

So far, so good! As you can see, the disk is available via its *Unix* disk name **ada4p1** and via the *GPT* label.
Let's label the disk at the "pure" GEOM level (not only at the GPT one):

```sh
# glabel label DATA4 ada4
# glabel list ada4
Geom name: ada4
Providers:
1. Name: label/DATA4
   Mediasize: 2147483136 (2.0G)
   Sectorsize: 512
   Mode: r0w0e0
   secoffset: 0
   offset: 0
   seclength: 4194303
   length: 2147483136
   index: 0
Consumers:
1. Name: ada4
   Mediasize: 2147483648 (2.0G)
   Sectorsize: 512
   Mode: r0w0e0
```

Now the disk is avaialble via another name, as well as the previous two ones:

```sh
# ls -l /dev/ada4p1 /dev/label/DATA4 /dev/diskid/DISK-VB97f8d8f5-8f70a2a4
crw-r-----  1 root  operator  0x74 Oct  2 16:41 /dev/ada4p1
crw-r-----  1 root  operator  0x76 Oct  2 16:41 /dev/diskid/DISK-VB97f8d8f5-8f70a2a4
crw-r-----  1 root  operator  0x75 Oct  2 16:41 /dev/label/DATA4
```

Until we start using the disk, the *GEOM wither* does not triggers, so let's mount the disk and see if
the disk is still available via all the defined names:


```sh
# mount /dev/ada4p1 /mnt/data4
# ls -l /dev/ada4p1 /dev/label/DATA4 /dev/diskid/DISK-VB97f8d8f5-8f70a2a4
ls: /dev/diskid/DISK-VB97f8d8f5-8f70a2a4: No such file or directory
ls: /dev/label/DATA4: No such file or directory
crw-r-----  1 root  operator  0x79 Oct  2 16:46 /dev/ada4p1
```

As you can see, once the disk is mounted, the other names are no more availables. They became again available once the diskis un-mounted:

```sh
# umount  /mnt/data4
# ls -l /dev/ada4p1 /dev/label/DATA4 /dev/diskid/DISK-VB97f8d8f5-8f70a2a4
crw-r-----  1 root  operator  0x79 Oct  2 16:46 /dev/ada4p1
crw-r-----  1 root  operator  0x74 Oct  2 16:48 /dev/diskid/DISK-VB97f8d8f5-8f70a2a4
crw-r-----  1 root  operator  0x68 Oct  2 16:48 /dev/label/DATA4
```

This **GEOM names appearing and disappearing is the consequence of *GEOM withering**. A GEOM can be used in three different ways:
- reading
- writing
- with exclusive access

The latter is the cause of GEOM withering: once a disk is mounted an exclusive lock is required to inform the other available GEOM providers
that the disk is *in-use*. GEOM therefore withers the other disk available names, that results in the names to disappear from the
```dev``` filesystem. For more information see ```g_wither_geom(9)``` and ```geom(4)``` ORPHANIZATION example.
