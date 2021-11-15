---
layout: post
title:  "OpenBSD and Lenovo Thinkpad T410 Wireless Firmware"
author: Luca Ferrari
tags:
- openbsd
permalink: /:year/:month/:day/:title.html
---
A weird way to get the wireless card to work on OpenBSD.

# OpenBSD and Lenovo Thinkpad T410 Wireless Firmware

I am trying to get my Lenovo Thinkpad T410 working with OpenBSD as a daily driver.
One problem I faced, is that the wifi card is not working properly: it is recognized during the installation procedure as `iwn0`, but the installer is not able to configure it properly.
<br/>
This would be a simple task to solve if I had ethernet connectivity, that I didn't!
<br/>
Since I was installing OpenBSD other mobile phone tethering, how to fix it?
<br/>
The solution was:
- download the proprietary firmware and place it into an USB stick;
- install without internet connectivity;
- extract the firmware from the USB stick and place it on the computer;
- configure the network adapter.

<br/>
It could be possible to install the firmware *before* the installation starts, by going to a shell, but unluckily I was unable to find out the USB stick on my laptop. I mean, it was recognized as `sd1` from `sysctl hw.disknames` (where `sd0` was the other USB stick I was running the installation image from), but I was unable to get partition information from such disk, probably because I had to create the device myself.
<br/>
I therefore decided to continue with the installation, then rebooted the machine and got to the command prompt.
<br/>
Once I plugged the USB stick, recognized as `sd0`, I ran `disklabel sd0` that shown that the partition I was looking for was the `i`. I then mounted it with `mount /dev/sd0i /mnt` and copied the firmware I downloaded from %  [http://firmware.openbsd.org/firmware/7.0/iwn-firmware-5.11p1.tgz](% wget http://firmware.openbsd.org/firmware/7.0/iwn-firmware-5.11p1.tgz){:target="__blank"} on the local disk. I made the tarball to add the files to `/etc/firmware`.
<br/>
I then configured `/etc/hostname.iwn0` and rebooted, and the computer became online!
