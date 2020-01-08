---
layout: post
title:  "How to prevent your system from boot: destroy your encrypted swap partition!"
author: Luca Ferrari
tags:
- linux
- swap
- cryptsetup
permalink: /:year/:month/:day/:title.html
---
Today I decided to do some room-keeping of my partitioned hard disk, resulting in being locked off my own system!

# How to prevent your system from boot: destroy your encrypted swap partition!

A years ago I installed a new SSD hard disk as my primary hard disk, keeping the slower but larger one as *secondary* for backups and *slow stuff*.
Instead of deleting all the partitions that were there, I simply reformatted the old disk partitions and mounted them in a nested schema to access all the disk space. However, as you can image, as things grow on a single filesystem, this approach was problematic.
<br/>
How to solve it?
<br/>
Simple:**purge all partitions and make the disk a one (or two) big partition one**.
<br/>
<br/>
This is as easy as taking a cup of coffee, and I've done it so many times I could do it asleep. And of course, **the only way to do it right is doing in production!**
<br/>
*After all, who cares about backups?**
<br/>
<br/>
However, while doing this sequence of *delete* and *create* partitions, the kernel got confused so not all my new partitions where available. That triggered a quick reboot of the machine, and here's when things start going nasty.
<br/>
I didn't checked my `/etc/fstab` file, that contained references to the old (no more existent) partitions, so the machine was unable to boot.


## To boot or not to boot ...

When I write "the machine was unable to boot" I mean that the machine was even unable to give me a root shell, and in fact I was stuck in a *busybox* shell. That is not very comfortable, at least to me, since I'm used to get a `root` shell and try to recover the system from there, while in this case I was running on an in-memory system.
<br/>
Even booting in single user mode provide me a more concrete shell.
<br/>
Luckily, it is quite hard to scare me, so I decided to mount those partitions I remember about in the effort to recover something.
<br/>
So, after having mount the root filesystem, I inspected the `/etc/fstab` file and commented out the nuked partitions. However, this did not suffice to boot the machine and I was looped in the same busybux shell.

## What was the problem?

After inspecting the `/etc/fstab` file in a deeper way, I guessed the problem was related to *swap*.
I removed a swap partition, encrypted, and `cryptsetup` (and friends) was unable to find it anymore. And the boot was hung waiting for the crypto-swap to be activated.
<br/>
The problem therefore was **having an encrypted swap partition that has been deleted from the disk partition table**. And, just to make things a little more complicated, I had two different encrypted swap partitions on different disks.
<br/>
<br/>
Anyway, I was trying to recover my computer starting in single user mode and being thrown to a pretty much useless *busybox* shell. At least, useless to me.
<br/>
Even juggling with `mount` and editing the system `/etc/fstab` did not solved the problem because the ram disk did have the encrypted swap partitions hard-coded into it.

## The Solution 

In order to fix the problem I downloaded (from another PC) an ISO image and booted the computer from the USB key.
Then, with a complete Linux system running, I opened a shell and did the following:
1) mount the `/` partition on `/mnt`, something like `mount /dev/sdb2 /mnt`;
2) bind devices and sys filesystems
```shell
# mount --bind /sys   /mnt/sys
# mount --bind /proc  /mnt/proc
# mount --bind /dev  /mnt/dev
```
3) change the root into the old filesystem `chroot /mnt` so that I got an almost running *jail*;
4) edit `/etc/fstab` to adjust the swap partitions in order to reflect the list of `blkid`;
5) edit `/etc/crypttab` removing (or adjusting) the status of the single maintaned swap partition;
6) run `update-initrd -u** to generate and overwrite the current init ramdisk;
7) reboot.

## Lesson Learned

**Keep always a bootable media around*, it can serve to get a running machine (including a web browser!) in minutes.
<br/>
<br/>
Again, I do believe FreeBSD is behaving more consistently when *panicying*, but it could be just me being biased towards such an operating system.


