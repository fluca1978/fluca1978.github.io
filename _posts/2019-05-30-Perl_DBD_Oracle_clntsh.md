---
layout: post
title:  "Incompatible libclntsh.so when compiling DBD::Oracle"
author: Luca Ferrari
tags:
- perl5

permalink: /:year/:month/:day/:title.html
---
Sometimes I need to connect from Perl to Oracle, and that is not always as simple as it sounds!

# Incompatible `libclntsh.so` when compiling `DBD::Oracle`

I tend to write my utilities with Perl, not because I want to make them obscure to my colleagues that don't know the language, but because Perl is often the right tool for the job. However, when I need to connect my utilities to Oracle, problems arise. 
<br/>
It is not that Perl is not good at connecting to Oracle, but that the *Oracle Instant Client* is not as easy to install as it sounds, at least in my opinion. And of course, `DBD::Oracle` depends on the Oracle Instant Client, so in order to get the latter working you need to get the former working too.
<br/>
On a server running a 64 bit Linux, I found sysadmins have installed 32 bit Oracle Instant Client:

```shell
$ file /oracle/instantclient/libclntsh.so.11.1
/oracle/instantclient/libclntsh.so.11.1: 
   ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), 
   dynamically linked, stripped

$ uname -p
x86_64
```

When I tried to compile `DBD::Oracle` I got an error message saying that **`ld` skipping incompatible `libclntsh.so`**, and that is because Perl was compiled as a 64 bit version, `DBD::Oracle` as a 64 library and in the middle the Instant Client was a 32 bit version.
<br/>
In order of disperation you can now:
- ask your sysadmin to install the 64 bit version of the Oracle Instant Client, and **this is the most correct approach**;
- install your own version of Oracle Instant Client;
- try to compile Perl and `DBD::Oracle` as a 32 bit version.
<br/>
For the sake of my utility, I decided to install a 64 bit of the instant client within my home directory, that is fine since (i) I was using `perlbrew` to run a more modern version of Perl and (ii) my utility was supposed to die once the work was done.
<br/>
<br/>
And so it is how I did the trick: placed my own private copy of the Oracle Instant Client in my home folder, set the `ORACLE_HOME` shell variable to point to such directory, and then installed the module via `cpanm`.
This gives me all the time to ask the sysadmins to install the 64 bit version without any rush!
