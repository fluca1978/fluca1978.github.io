---
layout: post
title:  "Eclipse and CVS"
author: Luca Ferrari
tags:
- oracle
- java
permalink: /:year/:month/:day/:title.html
---
Eclipse does not support any more CVS!

# Eclipse and CVS

This is not *news* in any way: **Eclipse IDE does not support CVS client anymore!**
<br/>
Why am I posting this here now? Well, because I discoverd this by myself when I installed a fresh Eclipse IDE. So far, I've kept my Eclipse IDE up-to-date and probably the CVS client was installed and never removed, so I did not notice the change.
But with a fresh install, the CVS client was no more available from the old *Install New Software* dialog!
<br/>
I searched, and I've found [the announcement for this change](https://www.eclipse.org/lists/cross-project-issues-dev/msg18643.html){:target="_blank"}.

<br/>
One solution that worked for me, running Eclipse IDE 2022-03, was to add this repository to the list of available download site: [https://download.eclipse.org/eclipse/updates/4.22](https://download.eclipse.org/eclipse/updates/4.22){:target="_blank"}.
<br/>
Then, selecting such repository, it is possible to download all the CVS tools for Eclipse.


## Is CVS too old for Eclipse?

Probably this is what Eclipse developers, and many other around the planet, think.
The problem is that there is still a lot of projects that run over CVS and its cousing SVN, so there is no way for a developer than use such tools to cooperate with other developers.
<br/>
I believe that in *the Git rush* the developers started to think that old-school version control systems are no worth anymore, but hey, just have a look at [OpenBSD](https://openbsd.org){:target="_blank"} to get an idea about how useful CVS can be!
