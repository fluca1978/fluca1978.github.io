---
layout: post
title:  "Plasma Vault: Importing a Volume"
author: Luca Ferrari
tags:
- kde
- planet-kde-org
- plasma
- plasma-vault
permalink: /:year/:month/:day/:title.html
---
Plasma Vault is a great applet for the KDE Plasma desktop environment, but lacks a way to import the volumes.

Plasma Vault: Importing a Volume
---

[Plasma Vault](https://github.com/KDE/plasma-vault){:target="_blank"} is a great applet for creating and managing encrypted volumes based, mainly, on [CryFS](https://www.cryfs.org/){:target="_blank"}. It is a much simpler system with regard to my most used [Veracrypt](https://www.veracrypt.fr/en/Home.html){:target="_blank"}, but it has a nice and good design, UI and **makes the user aware of the importance of scattering its data in crypto containers**, that according to me is important.
<br/>
One thing that, as far as I know, Plasma Vault is missing is the option to import a volume, for example from another location on the local storage, from another storage, or another machine.
<br/>
In this article I show you a way to force Plasma Vault to import another volume.
<br/>
<br/>
**WARNING: this is not a suggested way to import volumes, but so far it works, at least for me**. Be very careful with this technique since you could end up destroying your applet configuration and, therefore, the way to access your containers from the applet itself. However, don't panic, as far as you have your containers you will be able to mount them using the filesystem tools.


# The Plasma Vault Configuration File

Plasma Vault stores the configuration in the file `~/.config/plasmavault.rc` that is an INI style text file. There are two sections that needs arrangment in order to import a new volume:
1) a specific section that points to your new volume;
2) the `EncryptedDevices` section that *enables* every volume to appear in the applet list.

<br/>
Suppose you want to import the `~/crypto-vaults/crypto-cloned.enc` volume, that is based on CryFS. After having made a backup copy of the `~/.config/plasmavault.rc` file, edit it with your favourite editor and add a section for the new volume as follows:

<br/>
<br/>
```shell
[/home/luca/crypto-vaults/crypto-cloned.enc]
activities=
backend=cryfs
lastStatus=1
mountPoint=/home/luca/Vaults/crypto-cloned
name=crypto-cloned-import
offlineOnly=false
```
<br/>
<br/>

The import parts are:
- `[/home/luca/crypto-vaults/crypto-cloned.enc]`, this is the init of the section and points to the actual on disk encrypted volume;
- `backend=cryfs` this tells to Plasma Vault how to mount the encrypted file system, that is which backend *fuse* layer to use;
- `mountPoint=/home/luca/Vaults/crypto-cloned` this tells to the applet where to mount the filesystem, and it could be any place you like (assuming you are not mounting it on a network available place!);
- `name=crypto-cloned-import` this tells to the applet how the volume will be appear in the applet list.

<br/>
Then you need to add the above volume into the `EncryptedDevices` section:

<br/>
<br/>
```shell
[EncryptedDevices]
/home/luca/crypto-vaults/crypto-cloned.enc=true

```
<br/>
<br/>


Note that the entry has the same exact path as the one in square brackets or the device section.
Be sure to not remove any other entry already present in the section, as well as not to add another section.

# Final Result
You need to trigger a Plasma Vault applet reload for changes to take effect, for instance by logging out from Plasma.
Once the applet has received the changes, you are going to see the crypto volume listed in you applet UI.


<br/>
<br/>
<center>
<img src="/images/posts/plasma/plasma_vaults01.png" />
</center>
<br/>
<br/>
