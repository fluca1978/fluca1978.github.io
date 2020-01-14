---
layout: post
title:  "Watching new commits via IRC"
author: Luca Ferrari
tags:
- raku
- programming
permalink: /:year/:month/:day/:title.html
---
The Perl 6 documentation projects pushes commit information to the IRC channel `#perl6`, so that it is possible for both users and reviewers to have a glance at what is happening and how far the documentation is evolving.

# Watching new commits via IRC

I realized only a few weeks ago that the [Perl 6 documentation project](http://docs.perl6.org) provides backtracking information about the commits on the `#perl6` IRC channel on `freenode`. What is so special about that? Well, while I find it quite interesting as a way to announce new features, this is also a meter about how much does Perl 6 care about its own documentation. Having information about new commits makes people interested in reading the documentation and, most notably, in reviewing its content and therefore in improving it.


The following is a short example about a *push* I did:

```
<Geth> ¦ doc/master: 5 commits pushed by (Luca Ferrari)++  [11:04]
<Geth> ¦ doc/master: c01852ffc3 | Extended the explaination of $*GROUP and
       $*USER variables.
<Geth> ¦ doc/master: 50382c5937 | Add blank line before =item POD directive.
<Geth> ¦ doc/master: 0cfc6e57f6 | Little changes to explaination of $*PERL.
<Geth> ¦ doc/master: 9c24744b8b | More on $*KERNEL, $*VM, $*DISTRO.
<Geth> ¦ doc/master: 6ed3006eb5 | Fix link directive, resolves mistake from
       6fb710b8.
<Geth> ¦ doc/master: review:
       https://github.com/perl6/doc/compare/2decb0d2f2...6ed3006eb5
```
