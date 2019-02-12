---
layout: post
title:  "A NFS + NIS + autofs Story"
author: Luca Ferrari
tags:
- linux
- kubuntu
permalink: /:year/:month/:day/:title.html
---
While ending the [M.W. Lucas' book *"FreeBSD Mastery: Specialty Filesystems"*](https://mwl.io/nonfiction/os) I did a jump back in time to my very first work as sysadmin...

# A NFS + NIS + autofs Story

When I was young and green (as a sysadmin), I got a job at my local university as sysadmin for a couple of Unix labs. One of those, in particular, was running a mixed environment of Linux and Solaris machines. And it was running *inside-out*, at least for me: there was a Linux server serving other Linux and Solaris machines (well, I was expecting the great and luxury Solaris being the *king* and serving the others!).

<br/>

Anyway, the laboratory was used by several students who have the ability to log in on any of the machines and get back their home directories and stuff. **NFS to the rescue!** Yes, the Linux server was exporting its file system, including the home directories, via NFS, a well supported remote filesystem on both Unix and Linux. It was NFSv2, back then.

<br/>

Exporting the filesystem was just half of the work, or even a third of it: different operating systems could have little different details and executables, so the user was really getting a home directory bound to the platform he was logging into. That meant a home directory for Linux 32 bit, one for 64 bit, one for Sparc and so on. And of course, this was against the *sharing* the some stuff without any regard to the computer he was logging into, so I provided also a *universally shared* folder for each user, where he could put its stuff and found it on every machine.

<br/>

But users were not real Unix users, they were students, and students sometimes don't even spell their username correctly, so you cannot expect them to *mount** a directory in the right way.

<br/>

**Automounter, quick**: introducing the automounter. Automouter was, back then, a great piece of software. Long story short: it allows the user to reference a path of an *unmounted* file system getting the system mount it automatically on demand and releasing it once it is no more used. That looks great for home directories: once the user logs in his home directory is automatically mounted from the NFS remote server (along with the other directories exported for such user** and this ensures the user can effectively work on a remote home from pretty much anywhere.

<br/>
<br/>

**So far, this is the only sane usage of the automounter I've ever seen** and no, mounting removable media devices is not a good candidate for me (device rules like `devd` being a lot more usefu.
<br/>
**At the same time, is was not a cat's pijama**: if the user logs in and out from different machines quickly, he get the home mounted on several machines at the very same time, as well as logging into different machine at the same time, making therefore  hard to keep copies consistent. But after all, there were not enough phisical computers for all the students, so why a single student whould log in simultanously on more than one? (Exclude the case of that beautiful girl asking for you credential because she left hers on the yesterday bra...)

<br/>
<br/>

Having therefore all home and shared directories automaticall mounted sounded great, but having to edit the plumbing mess of files like **`auto_master`, `auto.master`, `auto_home`, `auto.home`** and so on (yes, Unix uses underscores, Linux uses dots) was something could make me falling asleep on the keyboard. And debugging was even worst, since the only thing you can do is a trial-and-fix-errors approach!

<br/>
<br/>

But before that, I had to synchronize accounts! **LDAP you fool!** Not so easy punk, **NIS was what I had**. And NIS is something that shown me that not Unix were created the same! Yes, Linux was serving NIS fine, but calling it `yp` (Yellow Pages), and Solaris was getting NIS not so well from a Linux machine, calling it, well, `nis`.
And I put all my efforts on not scratching `nsswitch.conf` using `vi` (something I hate with a passion) on every machine to make accounts working.

<br/>
<br/>

And here I am, almost 17 years later, thankful of not being anymore involved in such a working setup. And if you remember files like **`auto_master`, `auto.master`, `nsswitch.conf`** as well as `ypbind` and stuff like that, you are old at least as I am.
<br/>
**Wasn't it fun to make *real workstations* work in that way?**
