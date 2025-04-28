---
layout: post
title:  "Plasma Vault size overflow: how I switched back to Veracrypt!"
author: Luca Ferrari
tags:
- kde
- veracrypt
permalink: /:year/:month/:day/:title.html
---
A nasty error that prevented me to access my data!

# Plasma Vault size overflow: how I switched back to Veracrypt!

A lot of time ago, I started using *TrueCrypt*, a multiplatform application to store encrypted data on filesystem by means of FUSE and alike systems.

Then things changed, and I migrated to its successor, **VeraCrypt**.

So far, so good.

An year ago or so, I tried **Plasma Vaults**, encrypted containers embedded into the Plasma KDE GUI, implemented by means of *CryFS*.

However, I encountered pretty soon a problem: the size of my vaults was too big for CryFS to handle. I opened an [issue to report my problem](https://github.com/cryfs/cryfs/issues/463#issuecomment-2834607707){:target="_blank"} but, apart from a few comments of people having the same issue, there is no official workaround. In fact, comments are pointing out how removing or renaming the Plasma related stuff would fix the problem, but I was using `cryfs` directly, so no Plasma was involved in there.

That's why, at that time, I switched back to VeraCrypt as my main *userspace filesystem encryption tool*.
