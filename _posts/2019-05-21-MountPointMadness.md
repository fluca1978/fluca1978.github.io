---
layout: post
title:  "Mountpoint madness: udisks and why I love FreeBSD the most"
author: Luca Ferrari
tags:
- linux
- freebsd

permalink: /:year/:month/:day/:title.html
---
Where does Linux mounts removable media? It depends on a lot of factors, maybe too much...

# Mountpoint madness: udisks and why I love FreeBSD the most

This is not intended to be a flame post. Really!
<br/>
As you probably know I use Linux, *Kubuntu* for what it matters (and it does matter!) on all my main machines (I mean pretty much every one on which I develop), and FreeBSD on servers (whenever it is possible) and dedicated machines. It is my choice, it not has to be *the right choice*, but it is a choice. Also you should know I'm not quite happy with Ubuntu and its family, but so far it works for me and I'm quite used to the Ubuntu approach.
<br/>
Anyway, in order to see what is lying around, I tend to test here and there some distros, and so it turned out now I've a backup computer running Fedora.
<br/>
Today I plugged in the Fedora machine my USB stick, and found it mounted at `/run/media/luca/FLUCA`, that is `/run/media/$USERNAME/$LABEL` instead of what I was expecting, such as `/media/luca/FLUCA** how Kubuntu does.
<br/>
**Uhm...some `udev` rules in the middle? No way!**
<br/>
So I start searching here and there for a configuration file, or something `systemd` related that could trigger that strange mount point abuse, but without any success. Until I discovered that there is another responsible: **`udisks`**. Long (and not interesting) story short: a *D-BUS* daemon that enables device management from an User Interface like KDE, Gnome and you name your favourite.
<br/>
What the fuck, I suspect FreeBSD is a lot more simpler here: `devd(5)` (the corresponding partner of Linux `udev`) and that's all. After all, it is what is used even for FreeBSD jails!
<br/>
However, [what is even worst is that it seems `udisks` **hardcoded** the mount point path and Ubuntu has patched it](https://askubuntu.com/questions/214646/how-to-configure-the-default-automount-location) to move to a more reasonable place. Are we back in time and still hardcoding paths? After all, who am I to criticize some piece of software that I don't know and I would not been able to write decently?

<br/>
<br/>
Now, let's review the target mount points as they are described in the *File System Hierarchy FHS* version 3:
- [`/media`](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s11.html#purposeMediaMountPoint) is described as a place to mount removable media, like zip disks;
- [`/mnt`](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s12.html#purpose13) is described as a temporary mount point for system administrator. I often use the example of mounting an ISO image to extract some programs into the base system;
- [`/run`](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch03s15.html#runPurpose) is described as directory for containing variable run-time information, what it was `/var/run` when I was young! Moreover, the documentation states that such directory should not be writable by non administrator users.

<br/>
Therefore, for God's sake, what is the point of making `/run/media` a user-based mount point?
<br/>
What is the point in having a mount point hardcoded?
<br/>
Why using `udev` and `udisks` on the very same machine?
<br/>
<br/>
Seems sometimes Linux goes too far for what my poor brain can handle. And that brings again up why I do love FreeBSD the most: things tend to be always the same. Because it does not sound to me that these changes are *innovations*, rather they are, in my opinion, *revolutions*.
<br/>
<br/>
As a final disclaimer, I'm not an `udev` guru, not even a `systemd` one, not even a `devd` expert, but chances are I can set up `devd` in less more time (and with less errors) than handling the former two correctly!
