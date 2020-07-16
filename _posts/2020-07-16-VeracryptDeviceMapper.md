---
layout: post
title:  "Veracrypt, Linux and the Busy Device error"
author: Luca Ferrari
tags:
- linux
- veracrypt
permalink: /:year/:month/:day/:title.html
---
An annoying problem I found while using Veracrypt.

# Veracrypt, Linux and the Busy Device error

I'm a little paranoid about my personal data: I keep it out of the cloud as much as possible, that means they are on USB sticks and hard drives. Of course, this also means someone could steal my data, so I keep them encrypted with [Veracrypt](https://www.veracrypt.fr/en/Home.html){:target="_blank"}, a flexible and easy to use solution that allows me portability across multiple operating systems.
<br/>
<br/>
A couple of days ago I came into an annoying problem with Veracrypt: I was no more able to mount my favourite volumes because volumen number `2` was *busy*:

<br/>
<br/>
<center>
<img src="/images/posts/veracrypt/device_mapper_1.png" />
</center>
<br/>
<br/>

I thought that a reboot could fix the problem, but it did not. So I start to investigate a bit more.
Inspecting the  `/dev/mapper` folder I saw the root of the problem:

<br/>
<br/>
```shell
% sudo ls -l /dev/mapper
total 0
crw------- 1 root root 10, 236 lug 16 08:34 control
lrwxrwxrwx 1 root root       7 lug 16 08:34 cryptswap1 -> ../dm-0
lrwxrwxrwx 1 root root       7 lug 16 08:40 veracrypt2 -> ../dm-2


% sudo file /dev/dm-2 
/dev/dm-2: block special (253/2)
```
<br/>
<br/>

There's a dangling `veracrypt2` link to the special device `dm-2`. And there's also a dangling `dm-2` block device that should have evaporated!
<br/>
I tried then to fix the problem removing both the dangling link and the device file:


<br/>
<br/>
```shell
% sudo rm /dev/mapper/veracrypt2

% sudo rm /dev/dm-2
```
<br/>
<br/>

However, this did not fixed the problem, and the error remained the same.
<br/>
The problem was the device mapper has *cached* the block devices, so even if the block device was not anymore on the filesystem, it was there for the mapper, and the latter prevented its usage.
<br/>
A reboot fixed the problem.
<br/>
I'm pretty sure there's a simpler way to inform the device mapper to *reload* the devices, but I've no advanced tools installed on my desktop.


# Conclusions

In order to get the problem disappear you have to manually remove the link and the block device from under your `/dev` folder (*be careful, you could accidentally remove an in-use mapped device!*), and then the `device-mapper` has to be informed about the changes. In my case, a reboot fixed this last issue.
