---
layout: post
title:  "OpenBSD Disk UIDs"
author: Luca Ferrari
tags:
- openbsd
permalink: /:year/:month/:day/:title.html
---
Getting rid of diskNumber schema.

# OpenBSD Disk UIDs

OpenBSD can use *disk UIDs* as a way to uniquely identify a disk.
<br/>
The need for an *uncoupled from hardware* way to identify disks is a very old topic, and common to pretty much all the operating systems. The advantage of using a different schema from the hardware tree is that you are free to move your disk around, for example attaching it to another bus or controller, being sure that it will be recognized anyway without any problem.
<br/>
Being OpenBSD such a great operating system, it does support *disk UIDs* as many other Unix-like operating systems.

## What is mounted?

Let's start simple and inspect what is mounted on a common installation:

<br/>
<br/>
```shell
luca@puffy ~ % mount
/dev/wd0a on / type ffs (local)
/dev/wd0k on /home type ffs (local, nodev, nosuid)
/dev/wd0d on /tmp type ffs (local, nodev, nosuid)
/dev/wd0f on /usr type ffs (local, nodev)
/dev/wd0g on /usr/X11R6 type ffs (local, nodev)
/dev/wd0h on /usr/local type ffs (local, nodev, wxallowed)
/dev/wd0j on /usr/obj type ffs (local, nodev, nosuid)
/dev/wd0i on /usr/src type ffs (local, nodev, nosuid)
/dev/wd0e on /var type ffs (asynchronous, local, noatime, nodev, nosuid)

```
<br/>
<br/>

In the above, the hardware disk `wd0` is mounted. The `wd` driver is one of the commonly used in OpenBSD. Just to recap, OpenBSD disklabels are identified by letters, where:
- `a` means the root (on the first disk);
- `b` means the swap partition;
- `c` means the whole disk (and thus is never mounted as it is);
- `d` .. `j` are the remaining user-defined partitions.

<br/>

## How does it look `fstab` like?

Let's inspect the `/etc/fstab` file for the very same installation:

<br/>
<br/>
```shell
luca@puffy ~ % cat /etc/fstab
944d666344bea7ce.b none swap sw
944d666344bea7ce.a / ffs rw 1 1
944d666344bea7ce.k /home ffs rw,nodev,nosuid 1 2
944d666344bea7ce.d /tmp ffs rw,nodev,nosuid 1 2
944d666344bea7ce.f /usr ffs rw,nodev 1 2
944d666344bea7ce.g /usr/X11R6 ffs rw,nodev 1 2
944d666344bea7ce.h /usr/local ffs rw,wxallowed,nodev 1 2
944d666344bea7ce.j /usr/obj ffs rw,nodev,nosuid 1 2
944d666344bea7ce.i /usr/src ffs rw,nodev,nosuid 1 2
944d666344bea7ce.e /var ffs rw,nodev,nosuid,noatime,async 1 2
```
<br/>
<br/>

Where is `wd0` now? **That's the beauty of disk UIDs: no hardware name is specified in the `fstab` configuration**.
<br/>
In short, the number `944d666344bea7ce` is an alias for `wd0`, or better, is the **label of the disk** and is immutable with respect to where the disk will be attached.
<br/>
Note that the `b` (swap) partition is specified in `/etc/fstab` while, clearly, it cannot appear in the `mount` command output.


## From partition names to *duid*

Please note that there is a little difference in the naming schema when using device names and disk UIDs (namely, *duid*`). **A partition is always identified by a letter** (as described above) **but when using a device name the letter is "simply" appended to the devie name (e.g., `wd0` partition `i` means `wd0i`)**, on the other hand **in the case of disk UIDs the partition letter is prefixed by a dot (e.g., `944d666344bea7ce.i`)**.
<br/>
Also, it is interesting to note that, while a device name could require a full path to the device, such as `/dev/wd0i`, when using a disk UID there is no need of a device path. Or better, *there is no device path* with such a name, so you don't have to prefix the disk UID with a `/dev` path.

## Find out the disk UID

How can a human being find out the association between a device node and the label of the disk?
<br/>
The first method is to inspect the `hw.disknames` system control:

<br/>
<br/>
```shell
luca@puffy ~ % sysctl hw.disknames
hw.disknames=wd0:944d666344bea7ce,cd0:
```
<br/>
<br/>

This machine has two disks: 
- `wd0` is labeled as `944d666344bea7ce`;
- `cd0` is not labeled because there is no disk into the driver.

<br/>
<br/>
There is another way to inspect the UID of a disk, and it is by means of `disklabel`:


<br/>
<br/>
```shell
luca@puffy ~ % doas disklabel wd0
# /dev/rwd0c:
type: ESDI
disk: ESDI/IDE disk
label: VBOX HARDDISK   
duid: 944d666344bea7ce
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 2088
total sectors: 33554432
boundstart: 64
boundend: 33543720
drivedata: 0 

16 partitions:
#                size           offset  fstype [fsize bsize   cpg]
  a:           880288               64  4.2BSD   2048 16384  6822 # /
  b:          1310050           880352    swap                    # none
  c:         33554432                0  unused                    
  d:          1162688          2190432  4.2BSD   2048 16384  9011 # /tmp
  e:          1653888          3353120  4.2BSD   2048 16384 12921 # /var
  f:          4218208          5007008  4.2BSD   2048 16384 12960 # /usr
  g:          1130272          9225216  4.2BSD   2048 16384  8760 # /usr/X11R6
  h:          3816448         10355488  4.2BSD   2048 16384 12960 # /usr/local
  i:          2891616         14171936  4.2BSD   2048 16384 12960 # /usr/src
  j:         10944224         17063552  4.2BSD   2048 16384 12960 # /usr/obj
  k:          5535936         28007776  4.2BSD   2048 16384 12960 # /home

```
<br/>
<br/>

First of all note how the command automatically changes the device `wd0` to `rwd0c`.
<br/>
This is *historical*: `r` means *raw* device (i.e., not partitioned in some sense) and the `c` means the *whole device*. Therefore `r + wd0 + c` means *all the disk `wd0` without dealing with paritions*.
<br/>
Having stated that, you can see in the output of `disklabel` that there is a row `duid: 944d666344bea7ce` that prints the UID of the device.


## DUID <==> Device Name

Since the disk UID (*duid*) is the same as a device name, you can use whatever you want in your commands:


<br/>
<br/>
```shell
luca@puffy ~ % doas disklabel 944d666344bea7ce
# /dev/rwd0c:
type: ESDI
disk: ESDI/IDE disk
label: VBOX HARDDISK   
duid: 944d666344bea7ce
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 2088
total sectors: 33554432
boundstart: 64
boundend: 33543720
drivedata: 0 

16 partitions:
#                size           offset  fstype [fsize bsize   cpg]
  a:           880288               64  4.2BSD   2048 16384  6822 # /
  b:          1310050           880352    swap                    # none
  c:         33554432                0  unused                    
  d:          1162688          2190432  4.2BSD   2048 16384  9011 # /tmp
  e:          1653888          3353120  4.2BSD   2048 16384 12921 # /var
  f:          4218208          5007008  4.2BSD   2048 16384 12960 # /usr
  g:          1130272          9225216  4.2BSD   2048 16384  8760 # /usr/X11R6
  h:          3816448         10355488  4.2BSD   2048 16384 12960 # /usr/local
  i:          2891616         14171936  4.2BSD   2048 16384 12960 # /usr/src
  j:         10944224         17063552  4.2BSD   2048 16384 12960 # /usr/obj
  k:          5535936         28007776  4.2BSD   2048 16384 12960 # /home

```
<br/>
<br/>

That is the same as `disklabel wd0` (see above).
<br/>
The same is true for `mount`:

<br/>
<br/>
```shell
luca@puffy ~ % doas mount /dev/wd0j /usr/obj

luca@puffy ~ % doas umount /usr/obj

luca@puffy ~ % doas mount 944d666344bea7ce.j /usr/obj
```


# Conclusions

OpenBSD disk UIDs are simple and nice to use. While people tend to think OpenBSD disk and filesystem is old and without features, the storage stack is fully functional (as the rest of the operating system!).
