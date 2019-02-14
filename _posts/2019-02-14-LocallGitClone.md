---
layout: post
title:  "A script to locally clone git repository and link them"
author: Luca Ferrari
tags:
- git
- linux
permalink: /:year/:month/:day/:title.html
---
I tend to organize my `git` repositories so that I can keep them always with me. In this post I introduce a small shell script that helps me doing the boring stuff.

# A script to locally clone git repository and link them

I tend to keep al my git repository locally organized into the `~/git` folder. That is nice and useful, at least to me, since I always have a fixed starting point. Many of such repositories have online shared counterparts, like [Github](https://github.com/fluca1978) or [Gitlab](https://gitlab.com/fluca1978), some of them will have, and some will never have.

<br/>
<br/>
Sometimes I need to bring a few repositories with me on a travel, on a different computer, and so on. **USB stikcs** and hard disk to the rescue! But USB sticks died (quite often), and when it happens I have to clone again the repository. That's not to much work, but quite frankly I don't like the git default naming scheme: **origin is a bad name for me** and I tend to provide more meaningful names following pretty much the following rules:
- `gitlab` or `github` as default remote for a shared repository under my account;
- `upstream` for a shared repository that is not mine (meaning *dangerous zone!*);
- `pc` for a remote that points to my computer;
- `usb` for a remote that points to and usb stick.

<br/>
That means that once I need to clone out a repository I have to do all the remote renaming stuff (yes, that's what *defaults* are for!), and moreover I've to do the backlinking of the remote into the original repository (if not existant**. Too much stuff for my fingers, let's do a script to automate that.

<br/>
Introducing **`local-git-clone.sh`**, that you can find [here](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/sh/local-clone-git.sh).
<br/>

Let's see how it works: you must pass a few arguments for the setup and the script will do the cloning and renaming stuff.
Arguments are:
- the local path to the original repository;
- the name you want the origin be renamed to;
- the name you want the new clone to be mentioned as remote in the original repository.

Let's see it in action during all the phases, and let's consider the [pgenv](https://github.com/theory/pgenv) repository that I keep a copy of.

## Phase 1: initial cloning

```shell
% local-clone-git.sh ~/git/misc/PostgreSQL/pgenv pc usb         
I will clone
/home/luca/git/misc/PostgreSQL/pgenv
into local folder
/media/luca/FLUCA_PG/git
creating the remote ref
pc
into it and adding, if needed, the remote ref
usb
into the remote source repository.
Ok to proceed? (y/n)
y
Step 1: clone remote [/home/luca/git/misc/PostgreSQL/pgenv] into /media/luca/FLUCA_PG/git ... 
Cloning into 'pgenv'...
done.
cloned into /media/luca/FLUCA_PG/git/pgenv
```

The idea is that of cloning the local `pgenv` repository onto the USB stick. First of all the program asks for confirmation, since the cloning could take a while. The the program issues a regular `git clone` command to start all the work.

## Phase 2: Renaming the `origin` remote

This is quite simple: the script issue a `git remote rename` on the freshly cloned repository to assign it a different name.

```shell
Step 2: renaming the default origin remote for the new repository ... 
origin -> pc
```

The repository on my local machine will be seen as `pc` from the usb stick.

## Phase 3: Cloning all the remotes

Now that the new repository is set-up on the usb stick, let's clone the remotes (excluding the usb stick itself) from the original repository.

```shell
Step 3: cloning all the remotes from the original repository ...
Adding [github] -> [git@github.com:fluca1978/pgenv.git]
Adding [upstream] -> [git@github.com:theory/pgenv.git]
```

Now the usb stick repository has the very same remotes as the original one, so it is somehow *a repository on its own* on which I can work from.

## Phase 4: Linking back the fresh repository

The original repository does not know anything about the new clone, so let's add it:

```shell
Step 4: adding usb as remote to /home/luca/git/misc/PostgreSQL/pgenv ...
```

In the case the original repository has already a remote with the same name, this operation is skipped.
Now I can `fetch` from either repository the changes of the other one.

## Phase 5: Fetching all

Well, at this time, the clone is done. But sometimes I want to be able to switch to branches I've done only in certain repository even when working offline. So, let's fetch all!

```shell
Step 5: fetching all ...
Fetching pc
Fetching github
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Total 8 (delta 4), reused 4 (delta 4), pack-reused 4
Unpacking objects: 100% (8/8), done.
From github.com:fluca1978/pgenv
* [new branch]      auto_select_pl_languages -> github/auto_select_pl_languages
* [new branch]      availables-pg-versions   -> github/availables-pg-versions
* [new branch]      build-from-git           -> github/build-from-git
* [new branch]      cleanup_available        -> github/cleanup_available
* [new branch]      configuration            -> github/configuration
* [new branch]      configure_opts_cleanup   -> github/configure_opts_cleanup
* [new branch]      fix-pgenv-root           -> github/fix-pgenv-root
* [new branch]      fix-typos                -> github/fix-typos
* [new branch]      master                   -> github/master
* [new branch]      merges                   -> github/merges
* [new branch]      minor-changes            -> github/minor-changes
* [new branch]      multiple-versions        -> github/multiple-versions
* [new branch]      patching                 -> github/patching
* [new branch]      self-check               -> github/self-check
* [new branch]      stop-modes               -> github/stop-modes
* [new branch]      verbosity                -> github/verbosity
Fetching upstream
remote: Enumerating objects: 2, done.
remote: Counting objects: 100% (2/2), done.
remote: Total 4 (delta 2), reused 2 (delta 2), pack-reused 2
Unpacking objects: 100% (4/4), done.
From github.com:theory/pgenv
* [new branch]      master     -> upstream/master
All done! Enjoy the clone repository /media/luca/FLUCA_PG/git/pgenv !
```

And that's all!
Now the usb stick contains a perfect workable standalone repository configured the way I want!

I hope this idea can be helpful for someone else.
