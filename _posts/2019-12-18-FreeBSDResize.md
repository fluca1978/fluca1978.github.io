---
layout: post
title:  "Resize FreeBSD disks (and filesystems)"
author: Luca Ferrari
tags:
- freebsd


permalink: /:year/:month/:day/:title.html
---
Sometimes I run out of disk space on my virtual machines, and adding more space is not too difficult in FreeBSD.

# Resize FreeBSD disks (and filesystems)

So, it happens sometime, especially when dealing with upgrades, that I run out of space on my virtualized FreeBSD filesystems. I tend to use the Fast File System on such virtual disks, because I don't need the power of ZFS on virtual machine and, rather, I want to save as much memory as possible.
<br/>
<br/>
Now, how can I resize the disk and the filesystem?
<br/>
Luckily, this is explained very well [on the officiale FreeBSD handbook](https://www.freebsd.org/doc/handbook/disks-growing.html){:target=_blank}. If you are lucky enough to have to grow a non-root disk, unmounting and proceeding with the following steps is quite straighforward, but in my case I needed to increase the root filesystem, therefore I had to boot from an installation media.
<br/>
In my case I had the first disk, `ada0` with three partitions:
- `ada0p1` is the boot partition;
- `ada0p2` is the root filesystem partition;
- `ada0p3` is the swap space.

<br/>
It was something like this:

```shell
% gpart show ada0
=>      40  37748656  ada0  GPT  (15G)
        40      1024     1  freebsd-boot  (512K)
      1064  35651584     2  freebsd-ufs  (14G)
  35652648   2096048     3  freebsd-swap  (1.0G)
```

<br/>
If you resize the disk in your virtualization environment, you will find the free space at the end of the disk, so in order to make the root filesystem partition to grow, you have to destroy the swap partition and recreate it later.
<br/>
**Of course, depending on the value of your data, you should do an appropriate backup before doing this**!
<br/>
After having booted, and therefore **having the partition not mounted**, I did the following:

1. `gpart recover ada0` to make gpart not blame about a corrupted partition;
2. `gpart delete -i 3 ada0` to delete the third partition of the disk, that is the swap space;
3. `gpart resize -i 2 -s 17G` to resize the root partition;
4. `gpart add -t freebsd-swap -a 4K ada0` to add again the swap space;
5. `growfs /dev/ada0p2` to make the filesystem grow.

And, of course, *reboot** the machine.
<br/>
<br/>
**I love `gpart` and I love FreeBSD!**


## Wait a minute, what about the swap space?

In the [handbook](https://www.freebsd.org/doc/handbook/disks-growing.html){:target=_blank} you will find that there is the need, or at least the suggestion, to disable the swap area before proceeding. I did not do that because I was not using the swap partition, and the process was fine. However, if in doubt, disable the swap partition before deleting it and remember to reactivate it once finished.
