---
layout: post
title:  "Installing Raku on OpenBSD"
author: Luca Ferrari
tags:
- openbsd
- raku
permalink: /:year/:month/:day/:title.html
---
Installing Raku and `rakudo` on OpenBSD is really a matter of seconds!

# Installing Raku on OpenBSD

How hard could it be to install our favourite language on OpenBSD?
<br/>
It's just a matter of seconds, because **`raku` and `rakudo` are available as packages!**

<br/>
Therefore, it simply needs to be issued a `pkg_add` command:


<br/<
<br/>
```shell
% doas pkg_add rakudo
quirks-4.53 signed on 2021-11-05T21:25:13Z
rakudo-2021.02.1:libatomic_ops-7.6.10: ok
rakudo-2021.02.1:libuv-1.40.0: ok
rakudo-2021.02.1:moarvm-2021.02: ok
rakudo-2021.02.1:nqp-2021.02: ok
rakudo-2021.02.1: ok

% which raku
/usr/local/bin/raku
```
<br/>
<br/>

The fact that `raku` is available as a package surpiresed me, [since it is not available on FreeBSD](https://fluca1978.github.io/2020/01/14/RakuOnFreeBSD.html)!
<br/>

Unluckily, there is no package for `zef`, so in order to [install `zef`](https://github.com/ugexe/zef){:target="_blank"} you have to do some manual steps:


<br/>
<br/>
```shell
% git clone https://github.com/ugexe/zef.git
% cd zef

% raku -I. bin/zef install .
===> Testing: zef:ver<0.13.4>:auth<github:ugexe>:api<0>
===> Testing [OK] for zef:ver<0.13.4>:auth<github:ugexe>:api<0>
===> Installing: zef:ver<0.13.4>:auth<github:ugexe>:api<0>

1 bin/ script [zef] installed to:
/home/luca/.raku/bin
```
<br/>
<br/>

Installing `zef` can require a lot of time...
<br/>


Anyway, after `zef` is optionally installed, it is time to fire up `raku` and start doing stuff:


<br/>
<br/>
```shell
% raku
Welcome to 𝐑𝐚𝐤𝐮𝐝𝐨™ v2021.02.1.
Implementing the 𝐑𝐚𝐤𝐮™ programming language v6.d.
Built on MoarVM version 2021.02.

To exit type 'exit' or '^D'
> $*VM.say 
moar (2021.02)

```
<br/>
<br/>

but that's another story!
