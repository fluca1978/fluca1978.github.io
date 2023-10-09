---
layout: post
title:  "KDE Vault, CryFS, damaged filesystem and Why I Do Not Use It Anymore!"
author: Luca Ferrari
tags:
- kde
- plasma
- cryfs
permalink: /:year/:month/:day/:title.html
---
I was already thinking to go away from KDE Plasma Vaults, but what happened to me today made me.

# KDE Vault, CryFS, damaged filesystem and Why I Do Not Use It Anymore!

**TL;DR: my `CryFS` based Plasma Vault stop working with the error: `Deserialization failed - size overflow`**. I have no solution to this problem , but I [opened a ticket with all the details](https://github.com/cryfs/cryfs/issues/463){:target="_blank"}, so in rush go read it in the hope for a solution.




I usually never complain about KDE and Plasma: it is a rock solid eye-candy desktop with a very few bugs in official releases.
However, this is time for a rant!

I started using KDE Plasma Vaults a few weeks ago, and I was not really impressed.
Vaults are a set of *almost automatically configured* encrypted folders that Plasma helps you managing. Vaults can work with a set of different user-level filesystems like **CryFS** (the default one) [which is a very simple to use encrypted filesystem](https://www.cryfs.org/tutorial){:target="_blank"}.

In short, CryFS places encrypted data into a bunch of files on the local storage, and mounts such folder as a crypto-device in userland.
KDE Vaults are just a GUI wrapping the undelying technology, which in turn is based on *FUSE*.

I stated that I was not impressed, and the main reason was about perfomances. I placed into a single Vault a few of my personal `git` repositories, stuff that I use, for example, to configure my computer as *dot-files*. Well, suddenly everything was going slow: Emacs and `git` in general were not able to show the contents, and in particular `git` was claiming that `status` was taking too much time!

Therefore, I started to wonder to change the backend filesystem in order to see if it was better.

No matter that I've tried `gocryptfs` without any particular speed improvement, today **my Vault just died!**

Suddenly, the Plasma applet was not mounting the vault and was not reporting back any error.
I then opened a terminal and tried to mount the vault by hand, obtaining **`Deserialization failed - size overflow`**:


<br/>
<br/>
```shell
% cryfs ~/.local/share/plasma-vault/git.enc ~/tmp/mount
CryFS Version 0.10.2

Password:
Deriving encryption key (this can take some time)...done
[Log] [error] Crashed: Deserialization failed - size overflow
```
<br/>
<br/>


No matter how hard I tried, CryFS was unable to open and mount my storage.

*Luckily, I had backups and I was not risking any data*, but I suspect CryFS is still not a mature filesystem implementation and has not any recovery tool for these kind of situations.


My encrypted file system was about `4,5 GB` in size. Since I used it everyday, I can confirm that it worked just fine the day before and that nothing *huge* was added to cause the above mentioned problem.


I [opened a ticket with all the details](https://github.com/cryfs/cryfs/issues/463){:target="_blank"} in the hope this could help finding out the reason. I'm going to keep the encrypted stuff for a few weeks in the case there is the need for some experimentation on it.
Unluckily, CryFS does not seem to provide a lot of information in the logs nor in the console output.


# Why I was not satisfied with the KDE Plasma Vault

Very poor performances was the most notable problem. I was unable to do normal `git`	 operations on repositories of a few hundred of megabytes, and branching and merging was a mess.

Besides, having moved my *dot-files* under a Vault meant that I was unable to start even my (customized) shell before mounting the vault. This was not a problem, after all, but since I use `systemd` user scripts to start, for instance, Emacs as a server, having a vault meant all the services were not started automatically. Therefore, I had to mount the vault and manually start all my services, and this is boring.

That's why I was already thinking to move away from vaults, and the incident above, even if did not make me loose any data, was the key to stop using vaults.


# Conclusions

having KDE Plasma Vault to manage my encrypted storage is nice and useful, but so far the FUSE based backends seems a little to sloppy for my needs.
I'm using VeraCrypt (known as TrueCrypt) since ten years or more, and that is working much more better and in a more scalable and robust way, so I guess this will be my next choice even for local encryption.
