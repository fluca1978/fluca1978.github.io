---
layout: post
title:  "FreeBSD 13, KDE Plasma 5.22 and Lenovo Thinkpad T410"
author: Luca Ferrari
tags:
- freebsd
- kde
permalink: /:year/:month/:day/:title.html
---
Run the best desktop environment on one of my favourite operating systems.

# FreeBSD 13, KDE Plasma 5.22 and Lenovo Thinkpad T410

FreeBSD has always been one of my favourite operating systems.
<br/>
KDE Plasma has always been my favourite desktop, and quite frankly it is one of the oldest programs I run, meaning that I've run it for almost all my computer science career!
<br/>
I've decided to install FreeBSD 13 on my old Lenovo Thinkpad T410 laptop. I know it is a very old machine, and it runs an Intel i5 quadcore, first generation, but the machine is still worth using and can serve me very well even at professional training events.
<br/>
In this article I briefly show how to install KDE Plasma 5.22 on FreeBSD 13. *Warning: I'm not going to cover the whole installation of FreeBSD, the configuration of an user account, of `sudo`, and stuff like that!* I assume the machine has already a FreeBSD installed and can be configured to run KDE Plasma.
<br/>


The final result of the laptop running KDE Plasma can be seen in this short video:


<br/>
<br/>
<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/KyzslJQfp_U" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</center>

<br/>
<br/>
In the following, the main steps to get the same behaviour.

## Step 1: initial packages

In order to get KDE Plasma running we need to install some other packages. For reasons I don't know, the Plasma package does not install the X Window system automatically, probably because it can be bound to Wayland or other compositors. Therefore, there is the need to install `xorg` as a package.
<br/>
The `kde5` is the package to install the Plasma desktop, so there is the need to install, clearly, such package. Now, it is possible to install `kde5` before `xorg`, but you will not be able to run the login manager at boot, so in order to get the setup up and running, you need to install both.

<br/>
<br/>

``` shell
% sudo pkg install xorg
...
% sudo pkg install x11/kde5
...
```
<br/>
<br/>

In particular, the `x11/kde5` consists of a little more than 700 packages!
<br/>
KDE is somehow Linux centric, less than other desktops, but still tied to Linux. Like it or hate it. What this means, is that in order to be able to run KDE Plasma you have to *modify* your FreeBSD machine so that it can "mimic" a Linux interface to those pieces of KDE Plasma that require it.

## Step 2: changing `/etc/fstab`

The KDE Plasma requires the operating system to have the `/proc` filesystem, that is not mandatory in FreeBSD but allows Linux to expose a file based interface to the running processes. This can be easily achieved with a change in the `/etc/fstab` file that, on my machine, appears as follows:

<br/>
<br/>

``` shell
% cat /etc/fstab
# Device        Mountpoint      FStype  Options Dump    Pass#
/dev/ada0p2     /               ufs     rw      1       1
/dev/ada0p3     none            swap    sw      0       0


# for KDE Plasma
proc /proc   procfs rw 0 0
```
<br/>
<br/>

The interesting part is the last line: it enables the *procfs* filesystem and configure the system to mount it as `/proc`, where Linux exposes it.
<br/>
Now that the system is ready to cope with the *rpcfs* filesystem, let's install `sddm` as login manager.


## Step 3: install the `sddm` login manager

`sddm` is a good login manager that can be easily installed with a single command:


<br/>
<br/>

``` shell
% sudo pkg install x11/sddm
...
```
<br/>
<br/>

## Step 4: enable the login manager

In order for the display manager to start at boot, it is required to enable it as a service, so the `/etc/rc.conf` should have the following lines:


<br/>
<br/>

``` shell
# for KDE and SDDM
dbus_enable="YES"
sddm_enable="YES"

```
<br/>
<br/>


The first line enables *dbus*, an intra-application communication system. The second line enables `sddm` as a service to be started at boot.



## Step 5: reboot!

It is now time to reboot the machine and see if the login manager appears.



# Lenovo Thinkpad T410

Everything is already in place for using the KDE Plasma desktop. However, if the running device is a Lenovo Thinkpad T410, different things are not going to work. In particular, the system is not going to detect correctly the HDMI port (or better, the display port).
<br/>
In order to get it running, there is the need to enable the `i915` kernel module. This is achieved by placing a line in the `/etc/rc.conf`:

<br/>
<br/>

``` shell
kld_list="i915kms"
```
<br/>
<br/>

but the above will enable the module once the system has already started. In order to enable the module in the early booting stage, configure also the `/boot/loader.conf` so that is contains:

<br/>
<br/>

``` shell
915kms_load="YES"
```
<br/>
<br/>

The effect of the above will be see during the boot phase, with the display that changes the resolution while the loading process progresses.
<br/>
Now, even the display port will be correctly detected, or ehm, read further.

## The Display Port Problem (Video Port)

The display port, and therefore the external monitor, will work great if the monitor is detected at boot, that means if the cable is plugged before the computer starts. If the cable is plugged after the computer has already booted, there is somehow the need for a *rescan* of devices, and this can be found with the trick of:
- switching the console from graphical (~9~) to textual by pressing `ALT F1` (or another number before 9);
- switching back to the graphical console with `ALT F9`.
<br/>

In other words, the function keys on the Lenovo keyboward will not work, and [I've asked for help on forums without success](https://forums.freebsd.org/threads/thinkpad-t410-and-display-port.82562/){:target="_blank"}, so in th case someone knows a better solution, please jump in the discussion!


# Conclusions

Running KDE Plasma as a desktop on FreeBSD is surely possible, and quite frankly even simple these days.
<br/>
There is no reason to merge these two great products together!
