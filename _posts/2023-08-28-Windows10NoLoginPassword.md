---
layout: post
title:  "Windows 10: no password prompt at login (without using netplwiz)"
author: Luca Ferrari
tags:
- windows
permalink: /:year/:month/:day/:title.html
---
A way to avoid having the password prompt when the computer starts.

# Windows 10: no password prompt at login (without using netplwiz)

**WARNING: don't try this at home!**
<br/>
*Removing the login password is a very bad idea, especially if you don't have other ways to protect your computer or data!*

<br/>
<br/>
So why am I willing to explain this?
<br/>
I use a Windows 10 machine to develop (some) software, and that machine is a virtual one running on a Linux host. Therefore, I don't have any particular data on it, since my code lies on a remote source code repository and the machine is only used as a *compiler*.
<br/>
Having said that, starting the virtual machine and being prompted for a password at login is annoying.
<br/>
Searching around the web, you will find that a lot of posts suggest you to change the settings via `netplwiz`, but that did not work for me since I don't have the required options, that I suspect have been removed during some sort of Windows upgrade.
<br/>
However, luckily and nastily, **Windows 10 allows local users to have an empty password**, so the simplest solution is to edit your account profile and change the login password to a blank one. Windows will not complain at all, and at the next boot it will login the last user without asking for a password.
<br/>
<br/>
I am still in doubt if I like or not this kind of solution...
