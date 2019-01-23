---
layout: post
title:  "I'm starting to hate again (K)Ubuntu!"
author: Luca Ferrari
tags:
- linux
- kubuntu
permalink: /:year/:month/:day/:title.html
---
I'm starting to get uncomfortable with Linux in general, and Kubuntu in particular.

# I'm starting to hate again (K)Ubuntu!

I do use [Kubuntu](ghttp://kubuntu.org) on many of my desktop systems, and yesterday I have to fresh install a new system. Easy pal, or at least this what I was thinking.

I burned a fresh *Kubuntu 18.10* USB stick just to get stucked around the end of the installation with a generic installer error. Trying several times, burning the stick from scratch and trying again lead me to the conclusion that only the minimal install could finish. Ok, assuming it was a problem with my USB stick, I decided to proceed anyway; after all I can install software as soon as I can login!

## Networking is Horrible!

Having to configure a static IPv4 networking, I opened a shell and tried well know commands...just to discover that `ip` is now the only one command available! Now, I don't know such command a lot, but luckily I get online with the followings:

```shell
$ sudo ip addr add 192.168.1.200/24 dev eno1
$ sudo ip route add default via 192.168.1.7
$ sudo echo "nameserver 192.168.1.254" > /etc/resolv.conf
```

(please note I'm using totally invented IPv4 addresses)
That worked...unless `NetworkManager` decided to try again to configure the networking overriding my settings. Ok, calm down, wait a little longer and repeat.

Then let's edit `/etc/network.d/interfaces` to set the old well known content:

```shell
auto lo
iface lo inet loopback
auto eno1
iface eno1 inet static
address 192.168.1.201
netmask 255.255.255.0
gateway 192.168.1.7
dns-nameservers 192.168.1.254
```

and reboot!
But **that did not work anymore!** Now networking is managed by **netplan**, something I don't know a fuck about.
Ok, where have I been in the last n-years? I don't know, but **why do I need yet another text-based way of configuring my networking instead of using the well know files?**

So, to fix the problem, just edit `/etc/netplan/01-network-manager-all.yaml` to something that looks like:

```yaml
network:
version: 2
renderer: networkd
ethernets:
  eno1:
   dhcp4: no
   addresses: [192.168.1.201/24]
   gateway4: 192.168.1.7
   nameservers:
     search: [mydomain.com]
     addresses:
        - 192.168.1.254
``**

**Is it more readable than the old Unix-style text files? I really don't think so!**
Anyway, after having created such file, let's apply it:

```shell
$ netplan apply
```

and see how the *networking* is ... well ... *working*!
Then inspect the old friend `/etc/resolv.conf` just to see that, for some reason I totally don't understand, the `search` entry is updated while the `nameserver` is not. Then spend a couple of hours trying to figure out why nameserver resolution is working while `netplan` is not updating such file, and finally give up!

<br/>
<br/>
As a final amusement: I really don't see the point in having another file format (YAML) to configure networking, having then to rely on another application to instruments all other components around the system. It reminds me when OpenSolaris was using XML (!) to configure services.


## Home Encryption is now an optional?

During the installation process I was not asked to encrypt my home folder, even if I disabled auto-login.
I had therefore to do it by myself:
1) I created another user in the `sudo` group just to quickly provide root capabilities;
2) I logged in as the new user, and then run
```shell
$ sudo ecryptfs-migrate-home -u luca
```
without a backup copy, since I was working on a fresh (empty) home directory;
3) I followed the instructions on the screen to get a copy of the hash and to encrypt the swap.

Why is this procedure required so far? One cool thying about the modern *nix installations is that encryption comes for free from the beginning, and now I have to do it by myself?


### Get Back Encrypted Data

Even if I do regular backups (well, not so regular, but at least I do before such a major upgrade!), I would like to *diff* my new home with the previous one in order to check against missing bits. The problem was that the previous home folder was encrypted on another hard drive. I then did the following:
1) mounted all the partitions of the previous hard drive in a single mount point;
2) `chroot` to such directory;
3) use `ecrypts-recover-private` to do the trick:
```shell
# ecryptfs-recover-private /home/.ecryptfs/luca/.Private
INFO: Found [/home/.ecryptfs/luca/.Private].
Try to recover this directory? [Y/n]:
INFO: Found your wrapped-passphrase
Do you know your LOGIN passphrase? [Y/n]
INFO: Enter your LOGIN passphrase...
Passphrase:
Inserted auth tok with sig [20f1131ab0f295db] into the user session keyring
INFO: Success!  Private data mounted at [/tmp/ecryptfs.yGKKVIMa].
```

That allowed me to recover that last edited file I forgot to include in the backups!

## Java is ... I don't name it!

Java seems broken, or at least, a `JDK 8` from Oracle is not working anymore. I had to install the official Java package via `apt`, that is Java 11, and then configure the alternatives to run as Java 8. After that, I was able to run [Eclipse 2018.12](https://eclipse.org).


## Cups and the Hell of SMB printers

Another hard problem, or at least a problem that required me a lot of time to fix, was related to printing to remote SMB printers. I had to send prints to a Windows Server <something> that, in turn, prints on a network printer. Yeah, it's cumbersome, but it's.

Now, there was no way to make the printer working with `smbspool`, receiving always a `NT_STATUS_ACCESS_DENIED`, without any regard of including username/password/domain into the `DEVICE_URI`. What was nice, is that the very same URI was working before.

After almost one day of digging, I found that invoking `smbclient` with `--max-connection NT1` and issuing a `print` command worked, so I searched for a way to force `smbspool` to behave the same. No way at all!

`ipp` to the rescue: I decided then to try to print directly via `ipp` and it worked in a couple of minutes!

Fuck off!

## Conclusions

Being used to install, fetch my *dot-files* and start working again seamlessy, this major update/installation was a giant kick in the ass! It reminds me the times when I was switching from Red Hat to Debian, and all things were different. **I appreciate even more the BSD approach, where everything stays as simple as it needs to be.**
