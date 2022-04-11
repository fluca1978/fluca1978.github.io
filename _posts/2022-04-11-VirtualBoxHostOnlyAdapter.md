---
layout: post
title:  "Virtual Box 6.1 (or greater) and Host-Only-Adapter"
author: Luca Ferrari
tags:
- linux
- vitualbox
permalink: /:year/:month/:day/:title.html
---
A strange way to configure Host-Only Adapters in the new releases of Virtual Box.

# Virtual Box 6.1 (or greater) and Host-Only-Adapter

In VirtualBox there is the capability to setup a so called **Host-Only Adapter**, that is a network layer that machines can share. I usually use a *NAT-onlyu* interface to let the machine go out on the Intenet, and an *Host-Only Adapter* to let my host machine to connect into the virtual machines.
<br/>
<br/>
Host-only adapter are configured, in the GUI application, via the *File*, *Host Network Manager*:

<br/>
<br/>
<center>
<img src="images/posts/virtualbox/host_only_adapters.png" />
</center>
<br/>
<br/>

After an upgrade to VirtualBox `6.1.32` I tried to configure the above window, but it resulted aslways in an *error with access denied*.
<br/>
After digging a little, I found out that [there is now the need for some policy option](https://www.virtualbox.org/manual/ch06.html#network_hostonly){:target="_blank"} into a file named `/etc/vbox/network.conf`. In short, such file must contain the list of available address ranges, **otherwise Virtual Box will allow you insert only addresses in the range `192.168.56.0/24`**.
<br/>
The syntax of this file is awkward: **every line needs an asterisk on its beginning**, and this lead me to some errors in the beginning.
So, for instance, the following is my network configuration file:


<br/>
<br/>

``` shell
% cat /etc/vbox/networks.conf
* 192.168.222.0/24
```
<br/>
<br/>

And that allows me to decide the range of subnetworks to use without any error. Clearly, you need to reload the Virtual Box application.

```

```
