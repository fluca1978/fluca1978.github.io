---
layout: post
title:  "Emacs Magit Spin-off branches: how to forget pain when using git"
author: Luca Ferrari
tags:
- emacs
- git
permalink: /:year/:month/:day/:title.html
---
Magit is probably the best git-porcelain ever!

# Emacs Magit Spin-off branches: how to forget pain when using git

Who knows me can say that I tend to live into Emacs. Well, not as much as I would, but I've always an Emacs instance (or a few) running and ready to obey!
<br/>
Who knows Emacs, knows also that it provides **magit**: absolutely **the best git porcelain ever built**!
<br/>
<br/>

Magit has a very interesting feature: *spin off branches*.
<br/>
It happened to all of us: we do a few commits before realizing we are on the wrong branch (it could be `master` or a branch we were working before).
<br/>
How to fix it? Well, the workflow is to create another branch (that will include those commits), get back to the wrong branch and reset the number of commits you want to disappear.
<br/>
<br/>
Being magit the wonderful piece of code it is, it provides you a very handy shortcut: you create a spinoff branch.
A spin-off branch is a new branch that automatically resets its *ancestor* branch back. Back to...the last published commit.
<br/>
The [official documentation](https://magit.vc/manual/magit/Branch-Commands.html){:target="_blank"} explains it very well:

<br/>
<br/>

```
b s (magit-branch-spinoff)

    This command creates and checks out a new branch starting at and tracking the current branch. That branch in turn is reset to the last commit it shares with its upstream. If the current branch has no upstream or no unpushed commits, then the new branch is created anyway and the previously current branch is not touched.
```
<br/>
<br/>

I love Magit!
I love Emacs!
