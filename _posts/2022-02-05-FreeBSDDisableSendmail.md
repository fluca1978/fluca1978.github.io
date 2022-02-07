---
layout: post
title:  "Faster booting FreeBSD disabling sendmail-related stuff"
author: Luca Ferrari
tags:
- freebsd
permalink: /:year/:month/:day/:title.html
---
How to improve the boot speed time by disabling useless services.

# Faster booting FreeBSD disabling sendmail-related stuff

It does not happen often, but FreeBSD sometimes needs to get rebooted. Well, in particular a few of my machines that I use only as testing workbenchs, and thus are turned off when I don't need them!
<br/>
Anyway, my FreeBSD 13 machine was booting slow, spending a lot of time doing some `sendmail` related stuff, in particular what seemed to me a reverse DNS lookup.
<br/>
<br/>
**But I don't use `sendmail` nor any related service on that machine**, so why is that enabled?
<br/>
I did not have too much time to dig it, but apparently there are some services that are now turned on by default (or I have done some wrong selection at installation time).
<br/>
The fix is not hard neither simple: there are four different services to disable.
After all, the `/etc/rc.conf` file looks like:

<br/>
<br/>

``` shell
sendmail_enable="NO"
sendmail_submit_enable="NO"
sendmail_outbound_enable="NO"
sendmail_msp_queue_enable="NO"
```
<br/>
<br/>

I hope I turned accidentally on such services, because I don't like very much having a system that starts too much services, not only because of speed, but also because of security concerns.
<br/>
Surely, this is something I need to investigate on as soon as I've time.
