---
layout: post
title:  "Automatically load Putty SSH Agent (pageant) on Windows"
author: Luca Ferrari
tags:
- microsoft
- ssh
permalink: /:year/:month/:day/:title.html
---
How to start automatically an agent to manage the SSH keys.

# Automatically load Putty SSH Agent (pageant) on Windows

PuTTY is the de-facto standard way to connect via SSH from a Microsoft Windows machine.
Configuring ssh keys on Windows is not so simple, since loading them at boot time requires much more steps, according to me, than on the Linux/Unix side.

<br/>

In order to load automatically the keys, there is the need for the *PuTTY Authentication Agent* named `pageant`. I suspect the name of the executable is mispelled with regard to the name of the application, since it seems to be *P*uTTY *AGE*nt for *A*utenthication *NT* (which could be the famous version of Windows, I don't know).

<BR/>
Here's how I configure it, after having generated the SSH keys on the Windows machine.

## Step 1: Enable `pageant` at boot time

Open the file browser and manually edit the path so to enter into the `Users\AppData\Roaming\Microsoft\Windows\StartMenu\` and then find the folder for the automatic execution of applications once you log in. Copy a link to the `pageant` application from your PuTTY installation.


<br/>
<center>
<img src="/images/posts/windows/ssh_agent_3.png" />
</center>
<br/>


Now edit the link you just created, by means of right clicking it and selection *properties*, and find out the command that is executed. Append to the command the location of the key you want to be loaded (full path in double quotes).

<br/>
<center>
<img src="/images/posts/windows/ssh_agent_4.png" />
</center>
<br/>


## Step 2: Logout and ensure `pageant` is running

Reboot (or simply logout and login again) to ensure that `pageant` is running. After a few seconds from the login, you should see a window appearing and prompting you for the passphrase.
If that is the case, everything is setup correctly and you can see the icon of the application in the tray.


<br/>
<center>
<img src="/images/posts/windows/ssh_agent_2.png" />
</center>
<br/>
<br/>
<center>
<img src="/images/posts/windows/ssh_agent_1.png" />
</center>
<br/>
