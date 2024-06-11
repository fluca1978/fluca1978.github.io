---
layout: post
title:  "The Disaster that is the Release Upgrade in (K)Ubuntu"
author: Luca Ferrari
tags:
- kubuntu
permalink: /:year/:month/:day/:title.html
---
Another scary story...

# The Disaster that is the Release Upgrade in (K)Ubuntu

I love the Ubuntu set of platforms, even if recent directions are not really mines (e.g., snaps).

But love is a complicated feeling, and usually comes also with a degree of **hate**: *I hate the upgrading process in Ubuntu*!

Today I upgraded, meaning I upgraded the release version, my workstation. So far, so good, everything was running fine during the upgrade.

Then it comes time to reboot.
And nothing happens.

I mean, **nothing appears on the screen**: I had a totally black screen without the loading process, without the login interface, without nothing.
But I got it on a dual monitor setup, so it was as bad as twice!

`CTRL-ALT-F3` to the rescue! It dropped me in a terminal, from which I tried to fix the broken system.
Without any luck at all!

It turned out the problem was related to the NVidia drivers, that I had to remove and reinstall from scratch.
After this, the machine shown me something on the screen, and the system started to work again.

I know (K)Ubuntu warned me about third party software, but I guess there are tons of desktops out there running NVidia drivers that can fall into the same problem, so I cannot think this has not been battle tested. However, in my unlucky experience, **every time I fully upgrade a desktop running (K)Ubuntu, I have to plan 2-3 hours of repairing**.

And this is not only related to the screen problem: all my Python virtual env stopped working, so that I had to create new ones. It is not such a big problem, unless you have to install a lot of packages in the new virtual environment...

**Please Ubuntu, fix this damn pain in the ass that is the `dist-upgrade` process!**
