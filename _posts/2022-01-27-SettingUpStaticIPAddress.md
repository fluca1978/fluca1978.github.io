---
layout: post
title:  "Setting a static IP configuration: how hard can it be?"
author: Luca Ferrari
tags:
- freebsd
- linux
- openbsd
permalink: /:year/:month/:day/:title.html
---
I tend to hate bloated Linux distributions...

# Setting a static IP configuration: how hard can it be?

Sometimes, luckily quite rarely, I have to setup a new Linux machine from scratch, and it often involves making the machine behave with a static IP configuration (address, routing, hostname).
<br/><br/>
*How hard can it be?*
<br/><br/>

Well, it is actually harder than you may think!
<br/>
In Fedora Linux, for example, the *suggested* way is to use `nmcli`, that is a powerful command line tool to manage networking configuration. The problem is, it is too much verbose according to me.
As an example, here's how to configure a static ip address:

<br/>
<br/>

``` shell

```
<br/>
<br/>

What about setting the hostname? Well, in a bloating system we must have a command to do that too!
<br/>
And here comes `hostnamectl`.
<br/>
Therefore, two commands, too much options to remember, too much verbosity.

<br/>
<br/>
What about the others? Well, in FreeBSD you have the unique file `/etc/rc.conf` to edit for both the network and hostname configuration. And it is that simple: fire up your editor of choice and place the `hostname` variable and `ifconfig_<interface_name>` variable.
<br/>
In OpenBSD is simple enough too: `/etc/hostname.<interface_name>` for the network configuration and `/etc/myname` for the hostname. Those are two simple text files that can be edited in less than a minute.
<br/>
<br/>

It is not only a matter of habits and taste: in FreeBSD and OpenBSD the configuration has been there since, well, day one! It means it is not going to change that much, and you can quickly get an help from the very rich documentation. Moreover, using text files is simple enough for interactive and automated processing.
<br/>
And Linux was similar too!
<br/>
Back in the days, I remember configuring a whole Linux system by editing `/etc/networks` files or even the *simple `/etc/rc.conf.local` catch-all configuration script.
<br/>
Then a ton of networking applications, configuration and management systems took the road and you will be quickly sucked into your own distribution approach, that clearly is different from any other distribution out there. Even the same distribution changes approach over the time (Ubuntu and netplan, anyone?).
<br/>
There is nothing wrong with changes, and nothing wrong with evolution.
<br/>
But, does changing from plain text to YAML improve the situation? I don't think so.
Does changing from a command to another improve the situation? I doubt...
