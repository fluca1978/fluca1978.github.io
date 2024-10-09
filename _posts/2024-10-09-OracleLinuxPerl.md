---
layout: post
title:  "Avoid installing perl-core on Oracle Linux... it is just a matter of time!"
author: Luca Ferrari
tags:
- perl
- linux
- oracle
permalink: /:year/:month/:day/:title.html
---
Sooner or later, you will need more than the base package

# Avoid installing perl-core on Oracle Linux... it is just a matter of time!

I have to admit, after a lot of years running Perl, this was the first time I've seen a Linux machine without `cpan`.

Long story short: I needed to install a Perl application of mine on a machine, so as usual I needed also some libs (modules) to get installed. The machine was not for me to administer, and `perl` reported version `5.16` (circa year 2012)!

After a little searching for, and after confirmation on the IRC, the machine (based on Red Hat) has only the `perl` executable and not even the real core of Perl, packaged as `perl-core`. So I installed such package and started installing stuff via `cpan`, even if I like `cpanm` the most.

Everything was working, until I met `DateTime`, that was requiring a C compiler. And the machine was lacking a C compiler too!

Therefore, I had to install a compiler, and at this time, I switched to another approach: [perlbrew to the rescue!](https://perlbrew.pl){:target="_blank"}
Therefore, I installed `perlbrew`, then compiled a **reasonably up to date version of Perl** (i.e., `5.40.0`, the latest at the time of writing) and, after having install `cpanm`, started to pull all the required dependencies.

Another approach could have been to install all the Linux distribution packaged `perl-xxx` modules that I needed, but chances are that I would have to compile something in any case, and I like the idea of having a recent version of Perl at my fingertips.
