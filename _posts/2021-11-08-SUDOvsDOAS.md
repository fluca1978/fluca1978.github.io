---
layout: post
title:  "doas vs sudo"
author: Luca Ferrari
tags:
- openbsd
- freebsd
- sudo
- doas
permalink: /:year/:month/:day/:title.html
---
What are you using to gain privileges?

# `doas` vs `sudo`

Yesterday I saw a poll on the FreeBSD forums: a [survey](https://forums.freebsd.org/threads/sudo-or-doas.82795/page-2#post-540595){:target="_blank"} about the usage of `sudo` or `doas` to execute commands as a privileged user.
<br/>
So far, `doas` is winning as the "new" default choice.
<br/>
But what is `doas`?

## `doas` as a `sudo` replacement

`doas` is a command appeared in OpenBSD 5.8, so it is not that old (around six years old), as a replacement for `sudo`. Why? Well, `sudo` was not in *base*, that is the minimal installed system, and therefore it was required to install from packages. OpenBSD developers wanted to provide a tool that could be shipped in base, hence the birth of `doas`.
<br/>
But it was not only a problem of "where" the command was: `sudo` is a quite complex beast and provides a lot of features that, quite frankly, I suspect many users don't know or don't use at all. Having a complex piece of code requires a deep introspection to prevent bugs and holes, and being OpenBSD a **very accurate** operating system, this has reached a point that was not tolerable any more.


## `sudo` public exposure

According to me, `sudo` become popular when operating systems like Apple OS X and Ubuntu started to use it as a way to not let the normal user to log-in as the `root` user. Quite frankly, in the beginning, I was disappointed by this choice because, especially with Ubuntu and derivates, the `root` user was unchanged and no password was set at all for it. This made administration a little more complex, in my opinion.
<br/>
Today, a lot of systems use `sudo`, but this command has a lot of features that go over the "simple" task of granting privileges. For example, `sudo` allows for an host-aware configuration, so that a single configuration file can be shared across different machines in order to set different rules on all of them.
<br/>
So far, even if I've different machines to orchestrate, I've never used such feature.


## `doas` vs `sudo`

`doas` has a simpler approach than `sudo` to the same problem, and provides a more readable configuration. In fact, the syntax for the configuration reminds that of a firewall, in particular of `pf`, and is therefore simpler to read even if you don't know about the command.
<br/>
Of course, `doas` does support less options than `sudo`, but so far I never missed `sudo` on machines where I installed `doas`.


## Which to use?

Quite frankly, I tend to use `doas` as much as possible. The problem is that, on many platforms, the `doas` is exactly in the same position `sudo` is with regard to OpenBSD: it needs to be installed by packages.
<br/>
That's not a problem, because on certain systems (e.g., FreeBSD), both the commands are in packages.
<br/>
Therefore, use the one that fits better for you, and for me it is `doas`. The only thing that I don't like in `doas` is that it reminds me the *run as* way of Microsoft Windows...

# Conclusions

`doas` is an interesting and simpler way of achieving many of the `sudo` capabilities without having to deal with *huge* configuration files.
<br/>
The only problem I see, and I have up to today, is that sharing machines with colleagues is not as simple as they are used to `sudo` instead of `doas`. And having to bounce between the one or the other requires my fingers and my brain to stay focused!
