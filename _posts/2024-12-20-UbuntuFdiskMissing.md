---
layout: post
title:  "Ubuntu and the missing fdisk command"
author: Luca Ferrari
tags:
- linux
- ubuntu
permalink: /:year/:month/:day/:title.html
---
Is Ubuntu becoming more and more less usable?

# Ubuntu and the missing fdisk command

Today I was trying to prepare an USB key for my son, so I opened a terminal on one of my (K)Ubuntu systems and tried to execute the well known `fdisk` command.
The shell complained that there is no `fdisk` command. Hurray! Linux has switched totally to `gpart`!



Damn, no!


The problem is that Ubuntu does not consider `fdisk` an useful program, so it does not install by default.
Gosh, Linux is becoming more and more difficult to use for me!


Clearly, the solution is to install the `fdisk` package:


```
% aptitude search fdisk
p   acorn-fdisk                                          - partition editor for Acorn/RISC OS machines
p   amiga-fdisk-cross                                    - Partition editor for Amiga partitions (cross version)
p   fdisk                                                - collection of partitioning utilities
p   libfdisk-dev                                         - fdisk partitioning library - headers
i A libfdisk1                                            - fdisk partitioning library


% sudo apt install fdisk
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Starting pkgProblemResolver with broken count: 0
Starting 2 pkgProblemResolver with broken count: 0
Done
The following NEW packages will be installed:
  fdisk
0 upgraded, 1 newly installed, 0 to remove and 484 not upgraded.
Need to get 121 kB of archives.
After this operation, 401 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 fdisk amd64 2.39.3-9ubuntu6.1 [121 kB]
Fetched 121 kB in 1s (197 kB/s)
Selecting previously unselected package fdisk.
(Reading database ... 321291 files and directories currently installed.)
Preparing to unpack .../fdisk_2.39.3-9ubuntu6.1_amd64.deb ...
Unpacking fdisk (2.39.3-9ubuntu6.1) ...
Setting up fdisk (2.39.3-9ubuntu6.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Not building database; man-db/auto-update is not 'true'.


```



But for God's sake, why is `fdisk` not an utility anymore?
What's coming next, will us be supposed to install `ls`?


This is my last rant before Christmas...
