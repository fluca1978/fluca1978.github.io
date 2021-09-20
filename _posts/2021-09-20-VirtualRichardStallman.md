---
layout: post
title:  "(Virtual) Richard Stallman can inspect your computer!"
author: Luca Ferrari
tags:
- open source
- linux
permalink: /:year/:month/:day/:title.html
---
No way RMS will ever spy on you!

# (Virtual) Richard Stallman can inspect your computer!

Clearly, the tile is a joke: **there is no way RMS could and would spy on you and your computer**!
<br/>
Therefore, what is this post about?
<br/>
It is about a program named `vrms`, that stands for *Virtual RMS*. This is a software that has been originated on Debian and is now ported to other distributions, like CentOS (and other Red-Hat based), where it is called `vrms-rpm`.
<br/>
But what is `vrms` or its cousin `vrms-rpm`?
<br/>
It is a program that *inspect* the list of locally installed application packages to see which one are not blessed by RMS; in other words it provides you with a list of packages that have software licences that are not allowed by GNU.
<br/>
Running such software, as you can see, is not mandatory and does not provide any boost to your system and its software, nor it will stop your computer to work: it will simply inform you that *you can do better with regard to GNU software*.
<br/>
Running such program is a good and interesting experience!

## Running `vrms` on a CentOS

On CentOS the package is named `vrms-rmp` and can be installed from the official repositories by means of `yum` or `dnf`.
<br/>
Running it is as simple as:

<br/>
<br/>
```shell
% vrms-rpm --ascii --explain
897 pacchetti liberi (96.7%)
31 pacchetti non-liberi (3.3%)
 - centos-logos
   Licensed only for approved usage, see COPYING for details.
 - gobject-introspection
   GPLv2+, LGPLv2+, MIT
 - gobject-introspection-devel
   GPLv2+, LGPLv2+, MIT
 - iwl100-firmware
   Redistributable, no modification permitted
 - iwl1000-firmware
   Redistributable, no modification permitt
   ...
```
<br/>
<br/>

The `--ascii` option will print an ASCII-art image of RMS in the case your system is clean, or when it is too few GNU-compatible. Depending on your terminal, this can appear as garbage, so you can use the `--image` flag to show a "better drawing" of RMS (by means of escape sequences):


<br/>
<br/>
<center>
<img src="/images/posts/linux/vrms-2.png" />
</center>
<br/>
<br/>


The `--explain` option will print out a line that explains why a particular package has been considered "bad". For instance, the `centos-logos` is not allowed by GNU because it has a licence that limits the usage (`Licensed only for approved usage, see COPYING for details`).
<br/>
<br/>
There are different options, so you can use the command to consider "good" specific licences, as well as you can ask it to list all *free* or *non free* packages by means of `--list` option. For example, to see that free packages installed, you can use `--list free`:


<br/>
<br/>
```shell
% vrms-rpm --ascii --explain --list free
...
 - emacs-common 
   GPLv3+ and GFDL and BSD 
 - emacs-filesystem 
   GPLv3+ and CC0-1.0 
 - emacs-nox                              
   GPLv3+ and CC0-1.0    
   ...
```
<br/>
<br/>
Guess what? [Emacs](https://www.gnu.org/software/emacs/){:target="_blank"} is considered as free software!


## Tuning the licences

It is possible to define which licences are considered *good* for you: you can list every licence in a text file and use it as the default dictionary when scanning thru installed software.


<br/>
<br/>
```shell
% echo BSD  > good_licences.txt
% echo MIT >> good_licences.txt

% vrms-rpm --ascii --explain --licence-list ./good_licences.txt
...
```
<br/>
<br/>
The output of the command will include all the packages that are considered non free, according to your *freedom definition*. For instance, with the above configuration, even packages with `GPL` (in any form) will be considered as poor licensed packages.

<br/>
<br/>
<center>
<img src="/images/posts/linux/vrms-1.png" />
</center>
<br/>
<br/>

# Conclusions

Using `vrms` can help you understand how well your stack is open source, and which components could break your licensing.
<br/>
Similarly, it can help you find out inconsistencies and incompatibilities across the system you are going to prepare, ship and deploy, or simply use!
