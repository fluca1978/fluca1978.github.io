---
layout: post
title:  "CentOS 8: Failed to download metadata for repo 'appstream'"
author: Luca Ferrari
tags:
- raku
- perlweeklychallenge
permalink: /:year/:month/:day/:title.html
---
Switching from CentOS 8 to CentOS Stream.

# CentOS 8: Failed to download metadata for repo 'appstream'

I'm not a fan of CentOS, on the contrary, I kindly hate it!
<br/>
However, sometime you have to deal with such distributions, and in my case I was managing a CentOS 8 system, and I faced the well know and established harsh update policy that switched CentOS from its "base" branch to the *stream* one.
<br/>
The symptom was that any software installation resulted in an error like `Failed to download metadata for repo 'appstream'`.

<br/>
Dealing with that was simple, even if it took time: I needed to switch the distribution to the *stream* release. That took two commands:

<br/>
<br/>

``` shell
% sudo dnf --disablerepo '*' --enablerepo=extras swap centos-linux-repos centos-stream-repos
% sudo dnf distro-sync
% sudo reboot
```
<br/>
<br/>

On my **old and out of date** system, it required `560` packages to be upgraded.
The reboot is not mandatory, but clearly recommended.
