---
layout: post
title:  "My Story About Revision Control"
author: Luca Ferrari
tags:
- my story about
- git
- mercurial
- fossil
- svn
- programming
permalink: /:year/:month/:day/:title.html
---
How did I ever met a revision control system and how I progressively approached it.

# My Story About Revision Control

A **revision control system** is something every developer should be aware of, and most notably **every developer should use**.
Interestingly, or better, awkwardly, it is something nobody teachs you in schools and universisties, so chances are you are not going to face a revision control system until it is too late (i.e., until you have to use it).
Luckily, the modern systems are enough spread so that they are pretty much well known, thanks to projects like [Github](https://github.com) and alike, but back in the days I was a young developer [Sourceforge](http://sourceforge.net) was the big thing!


## Backup is not a revision control!
Back in the first days of my activity as a developer, I already understood having a copy of a working source tree was a great value for keeping a *stable* version laying around. Because even if you change only a single line, **you do know that a single line change can break everything!**
However, while keeping a backup copy was a good thing, a single copy was not a guarantee I can go back in time to more than a version.

*Easy pal: just increase the backup frequency!* And so the solution was to prepare ad-hoc scripts to do a fulle source tree backup, scheduling such backups first at login/logout, and then via `cron(1)` every hour or so. Of course, this was not an efficient solution, but for a quite small source tree it did work and provide me a good solution to get back in time to *last well known to be stable* version of the source tree.

## A shared folder is not a collaborative space!
The need for collaborate with others revealed the first limit of the above backup solution: how to share the code with other developers?

**Being an ignorant developer** (thank you University for not teaching me that!) the only solution to share code seemed to be the same adopted in offices to share documents, and the only one comemrcial software companies were selling to people: **shared folders**.

At least I was regularly mirroring the shared folder on my own computer, backing-up regularly the source tree, and keeping track of changes.


## Entering CVS
While performing my PhD, I got into the [Aglets](http://aglets.sourceforge.net/) project (well, I did took the leadership since the project was abandoned). That was the first time I had to deal with a revision control, and back in those days, [Sourceforge](http://sourceforge.net) was using **CVS**.

Long story short: my mind was simply unable to handle revision control in such way! It is not that I believe CVS is a bad revision control system, on the other hand I do appreciate a lot of people being able to use it with success. Th real fact was that I was not able to better understand the concepts behind a checkout, and the first days I was simply doing a *fresh checkout every time I was needing to editing the source tree*.

Not very smart, but it was totally my fault!

Then I met (only thru email) a friend, Thomas Erlea, that helped me a lot understanding the basic concepts behind a revision control system, and so I was able to perform basilar tasks within the working tree. The awareness of operations performed by a revision control system totally opened my mind so that I needed to have revision control pretty much everywhere.

Unluckily, back in those days, distributed revision control system were something really hard to achieve and work with, not globally famous as they are today, and so I needed my old-school backup scripts in my toolbox for another long period.

And I have to admit also that, after having dealt with CVS, I did teach it to my tutors (please, read again, not students, tutors!) and tried to convince them to switch to a revision control system, failing.

# Entering Subversion
At work I was in charge of managing a much more huge source tree, with a lot of different stuff within it and more collaborators. Therefore, I took the decision to adopt what appeared to be the next big thing: *Subversion*.

I did configure a whole server, with different projects, ad-hoc *hooks* and was very happy with such configuration.

This experience revealed me that a lot of other developers did ignore the great value of a revision control system, in particular the ability to *merge* differente copies and keep track about the history of changes. It seemed to me that the old-school *comment out not working code* and *place a comment tag on the top of the file with the changes* was the preferred way to manage history. And quite frankly, it still is, at least when I have to walk thru some other people code.

I did some research and found [SVK](https://en.wikipedia.org/wiki/SVK), a set of tools to wrap Subversion repositories making them distributed. That was not really usable at the time, at least for me and my team, and I don't know how far the project went.

# Going distributed: Git
Subversion did have the same limitation of CVS: it required a central server to work as main repository, and you had to work connected to it in order to diff, commit, and so on. That resulted in a whole mess while other developers in my team were working at home during weekends, commit all the changes in a bulk at monday morning. What was the problem? They did forget the reason they were committing, or better, what did they changed, and so they were producing a mess of *single huge commits with no correct message at all* (like `Weekend update`).

When I first heard about [Git](https://git-scm.com/) I thought it was a real cool idea to be able to work in a *private offline space*, being able to switch other the main repo when possible and merge changes.
Moreover, that finally solved my problem to be able to carry on a revision control system everywhere (ok, there was the solution to either run a server on my linux machine or use `rcs`, but none was appealing to me).

The problem was I could not force the whole developer team to switch other [Git](https://git-scm.com/), and at the very same time I wanted to get a good knowledge about such system. Therefore I started to use [Git](https://git-scm.com/) over the company-wide Subversion, being able to work offline and merge back changes. I have to admit it was not funny at all, and it could be because I was new to Git and the tool was not as mature as it is today. But that allowed me to learn Git, so that I started to use it for any project I started from scratch, including documentation.

# Mercurial and Fossil
At the same time Git became famous, another project very similar took credits: [Mercurial (hg)](https://www.mercurial-scm.org/). Apart from some implementation details, the two were very similar in both approach and terminology (of course because they shared a common ancestor).
In order to get knowledge with Mercurial I knew the only way was to actively using it, so I switched a couple of Git projects of mine to Mercurial.

That means I was simultanously using CVS, SVN, Git and Mercurial. Nothing special, but for someone who did not know what a revision control system was, it was surely a forward jump.

As time went on, I started to prefer Git the most, switching all my main projects to Git, thanks also to services like [Github](https://github.com/fluca1978) and [Magit](https://magit.vc/), therefore much more due to the *porcelain* and not the *plumbing*!

Years later, I believe it was in late 2013, I found [Fossil](https://www.fossil-scm.org/index.html/doc/trunk/www/index.wiki), that I thougth it was really similar to Git (or Mercurial), while it was not. In order to learn that too, I converted a medium size project from Git to Fossil and worked with it for a couple of years. I have to say it is a very interesting distributed revision control, and I use it also today for personal small projects.

# What about today?
Today I do **use Git the most, and that's mainly because my personal integration with Emacs and the great porcelain [Magit](https://magit.vc/)**.

My second choice is [Fossil](https://www.fossil-scm.org/index.html/doc/trunk/www/index.wiki), while I have pretty much discarded [Mercurial](https://www.mercurial-scm.org/). It is not that Mercurial cannot work well and fast, it is just that I don't see the point in keeping myself up-to-date with a Git-like system as it is. Moreover, while some Git tool are based on Perl, Mercurial is based on Python and this is another good excuse to justify my choice, even if I would keep that one as a very last resort while in a beer-talk.

I never touched Subversion again outside my previous employeers, and I have to use CVS where I currently work.
