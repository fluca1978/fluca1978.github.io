---
layout: post
title:  "Damn! I locked myself out of my new Linux System!"
author: Luca Ferrari
tags:
- kde
- linux
- sudo
permalink: /:year/:month/:day/:title.html
---
What a mess I did while installing a new system!

# Damn! I locked myself out of my new Linux System!

I was re-installing my Linux laptop, using Fedora KDE spin (if that matters).
For various reasons, I haven't noticed that the main account, the one used for myself, was created with an incorrect username of `lferrariluca` instead of the usual and simple `luca` I use.
And of course, as usual in these automated installations, the `root` account was disabled.

When installation finished, and I rebooted my machine, I fired up my terminal just to find out the wrong username placed into the system.

Ok, time to fix this, **how hard can it be?**

Well, it turned out that, while it is still a simple task, it was not so simple as it seemed.
There is a correct way of doing this, and a wrong one.

**The correct way is to use `usermod`**, but I'm used to FreeBSD `pw`, so in a rush I was not thinking about this tool and, having yet no internet connection, I had to do this by myself.

So I edited `/etc/passwd` and `/etc/shadow` to rename the account into the right string. And bam! I got locked out, because I had no chance to use `sudo(1)` anymore. Clearly I forgot to edit the `/etc/group` file too, and since I closed the privileged session, I was no more able to edit anything.

Luckily, I still had the USB stick to start a rescue session, so I had to boot a live system, mount the root filesystem (that now is stored literally under a `/root` directory on the partition) and edit `/etc/group` within it. At least, Fedora uses the well established `sudo` rules where `wheel` members can become superusers, so it turned out that changing the `wheel` entry in `/etc/group` fixed everything.

This is a great trip back in the days where sysadmins were doing all this stuff by hands, and clearly nothing was in sync!
