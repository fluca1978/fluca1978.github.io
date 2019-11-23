---
layout: post
title:  "WIFI & Captive Portals: a Bad Idea!"
author: Luca Ferrari
tags:
- linux
- wifi
- captive


permalink: /:year/:month/:day/:title.html
---
No way I'm going to connect to an hot-spot that is basing its own security only on a captive portal!

# WIFI & Captive Portals: a Bad Idea!

In this days I'm a guest at a foreign university for an european project, and I've the proof that *system administrators* are every day in a rush to provide services, even in an unsecure way.
<br/>
The story goes like this: they provide me the `ESSID` of the WiFi network and also the username and password to connect. Uhm, why a *username*?
<br/>
The colleague near to me explained that as soon as I connect to the network a web page will pop up asking for a username and a password. **Captive Portal** on its way!
<br/>
What is the problem with this setup?
<br/>
Did you notice I haven't written about an `PSA` key? **The WiFi network is running on plain**!
<br/>
<br/>
I'm sorry pal, having been on the other side of the desk and having deployed captive portals (thanks to [pfSense](http://pfsense.org)), **I'm not going to connect my laptop to tan open network**!
<br/>
No way!
<br/>
*I will go for tethering instead*, which is a much more secure way of connecting my laptop instead of having a plain hot spot.
<br/>
<br/>
Am I against captive portals? No, I really like captive portals and I think they are a very smart way to monitor and control the traffic of users, at least to avoid automatic traffic generation (or limit it), but that's a different discussion.
<br/>
I'm against open hot spot, and quite frankly if you believe a captive portal will provide you some security, you have already lost.

## The real problem

I think that the real problem is that sysadmins are not knowing what they are delivering or they are forced to deliver a wrong stack of technlogies.
<br/>
Captive portals are a great way to let logged in user to route outside the network, but they are not a way to control and manage user permissions, nor to control what they do within the network.
<br/>
Once I get connected to an non-protected WiFi hotspot I'm within the (local) network. So while I'm not supposed to get routes for the outside world, I've already a route within your network segment, so I can start trolling around and see what you have.
<br/>
Moreover, the captive portal is often tied and served by the same appliance that is providing the hot-spot, that is once I kown to which WiFi antenna I'm connected, chances are I can discover were the captive portal is too. Is this bad? Apparently not, and I don't think it is too bad, but I think that knowing what is serving the captive portal will provide you some information about the appliance, and then you get the point...

## The solution

There is only **A-One Solution**: use protected WiFis.
<br/>
You are going to share the passphrase with your guest in any way, after all, so a captive protal over a non-protected WiFi is just giving you the illusion of safety.
<br/>
And if you really want to use a captive portal, do it the right manner: serve it as another protection layer over your network, not as the only protection layer!
