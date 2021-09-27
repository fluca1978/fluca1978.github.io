---
layout: post
title:  "A glance at Arch Linux (and I'm not happy with it!)" 
author: Luca Ferrari
tags:
- linux
- arch
permalink: /:year/:month/:day/:title.html
---
I tried Arch Linux, and throw it away really soon!

# A glace at Arch Linux (and I'm not happy with it!)

So, I finally took some time to try [Arch Linux](https://archlinux.org/){:target="_blank"}, a distribution that has a big momentum and is often accomunated to *being for hackers*.
<br/>
I have to say: I installed, booted, and decided it is not what I want for me.
<br/>
And it's not because of the installer, no way. It is because **I don't get the feeling of being comfortable using it**.
<br/>
If you find yourself at home using Arch Linux, please continue. 
<br/>
If you believe I'm not enough "hacker" to use it, please go browse away.
<br/>
If you feel there is something less than awesome with Arch, please continue reading.

## Look ma, I'm an hacker!

I often listen to podcasts and video tutorial that tell people how Arch Linux is great due to its basic command-line installer: "it will teach you how Linux works!".
<br/>
True.
<br/>
But if you have being around the Unix world for quite a long time, there is no hype here. I mean, after all the advatanges of *Anaconda*, *Calamaris*, and name another few, was that the sysadmin did not have anymore to type a quite long and difficult set of commands to format the hard drive, mount it, copy the software on it, `chroot` into it, and so on. Quite frankly, I've done it enough times in the past, and today I don't care really much about the installation process.
<br/>
Clearly, this is great if you need or want to fully customize your installation process, but in the era of the virtualization, it sounds to me a lot faster to snapshot a virtual machine than to produce a fully customized installation script. *Even if I feel more comfortable with the latter!*
<br/>
FreeBSD, and in general, all other BSDs, have basic installars, and can drop you into a shell to do whatever you want to your system during installation. A common case is to build a software RAID mirror before the system actually installs.
<br/>
Therefore, in the long run, I don't feel much impressed by the command line installer, and surely this is not what can push me towards or backwards Arch Linux!

## The Installation Guide

Arch Linux has a very complete wiki, and the [installation guide](https://wiki.archlinux.org/title/Installation_guide#Set_the_keyboard_layout){:target="_blank"} is a glorious resource.
<br/>
However, it looks to me not complete as expected. For example, there is no room for setting a boot loader, rather it just tells you "install a bootloader", leaving the user to research which one to install and how to.
<br/>
After all, the installation guide just list a set of commands to almost-blindly repeat on your installation to get it bootable.
<br/>
I believe there is a great difference between *being short* and *being complete**, and this wiki page does not look to me as it is complete.
<br/>
Moreover, the list of software suggested to be installed is really a skeleton part, and while I appreciate it, probably the user should have a couple of suggestions for extra software to install on the barebone system before it reboots.


## The Documentation

While the Arch Wiki is great, after booting a freshly installed system a bad surprise happened to me:


<br/>
<br/>
<center>
<img src="/images/posts/linux/arch/arch_man.png" />
</center>
<br/>
<br/>

**The `man` is not installed?** 
<br/>
How am I supposed to learn how to interact on a system without `man` installed?
<br/>
I can search for within the wiki, but that's not either a Unix approach, nor an "hacky" way. The system must be self-documented to let the administrator to work with it even in the dramatic situation where no internet is available!
<br/>
I'm used to BSD systems here, and that's a very good point of thos systems.

## Pacman

The beloved package manager [pacman](https://archlinux.org/pacman/pacman.8.html){:target="_blank"}.
<br/>
I'm sure it is great, but why it is asking me to use `-S` at every time? And why this option is not explained in the base systsem lacking of the `man`?
<br/>
I mean, if the `-S` *sync* option is the preferred one, why is not enabled by default?


# Conclusions

As you can see, I don't have any solid point against Arch Linux. It simply does not make me feel comfortable as a BSD system does.
<br/>
That's why I'm not really interested in trying it unless I have enough spare time.
