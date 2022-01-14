---
layout: post
title:  "fdisk scripts"
author: Luca Ferrari
tags:
- linux
permalink: /:year/:month/:day/:title.html
---
fdisk has a minimal support for scripts, that can help automating repetitive tasks.


# `fdisk` scripts

Low level partitioning tools such as `gpart`, `disklabel` and alike do support automation thru scripts, that is a set of commands to execute.
<br/>
But what about Linux's `fdisk` implementation?
<br/>
Well, it turned out that it does support scripts too.
<br/>
I happened to renew my set of USB sticks, and I wanted them all to be identical not only on an hardware basis, but also on a partitioning setup. Having to repeat all the low level `fdisk` operations by hand made me think about scripts, and so I read the man page to find out it can be done.
<br/>
Not having any experience with `partx`, I decided to solve the problem creating a first partition scheme, and then cloning it on all other devices.

## Creating a script, the quick way

Once you are inside `fdisk`, having created a partition scheme, you can **use the `O` command** to output the commands on a text file. The program will ask you for the file to produce, and the result will be something like the following:


<br/>
<br/>

``` shell
% cat keys.txt
label: dos
label-id: 0x0a186eaf
device: /dev/sdc
unit: sectors

/dev/sdc1 : start=        2048, size=    19494912, type=b
/dev/sdc2 : start=    19496960, size=    41943040, type=b

```
<br/>
<br/>

It clearly reminds a `disklabel` file.

## Using a script

In order to re-use a script, you need to enter `fdisk` for the device, and then **use the command `I` to make `fidks` read the script**.


<br/>
<br/>

``` shell
% sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): I

Enter script file name: keys.txt

Created a new DOS disklabel with disk identifier 0x0a186eaf.
Created a new partition 1 of type 'W95 FAT32' and of size 9,3 GiB.
Created a new partition 2 of type 'W95 FAT32' and of size 20 GiB.
Script successfully applied.

Command (m for help): p
Disk /dev/sdc: 29,3 GiB, 31457280000 bytes, 61440000 sectors
Disk model: PHILIPS
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0a186eaf

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sdc1           2048 19496959 19494912  9,3G  b W95 FAT32
/dev/sdc2       19496960 61439999 41943040   20G  b W95 FAT32

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Re-reading the partition table failed.: Device or resource busy

The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).


```
<br/>
<br/>

The `p` command has been issued only to prove that the partitions have been created.


## `partx`, `parted` and `fdisk` friends

As far as I understand, Linux's `fdisk` is based on `partx`, and there are a few commands to amnipulate partitions from a kernel point ogf view, like `addpart` and `delpart`.
<br/>
The tool that seems to me more likely `disklabel` or `gpart` is `parted`.
