---
layout: post
title:  "Sqitch and Sqitchers"
author: Luca Ferrari
tags:
- sqitch
- planet-postgresql-org
- itpug
permalink: /:year/:month/:day/:title.html
---
Sqitch has nothing particular to do with PostgreSQL, except it does support our beloved database!


# Sqitch and Sqitchers

Long story short: **if you are not using [`sqitch`](https://sqitch.org/) you should give it a look**.
<br/>
<br/>
[`sqitch`](https://sqitch.org/) does not ties itself to *only* PostgreSQL, but it does support a lot of relational engines. However, if you want to know how to start using Sqitch over PostgreSQL go read the excellent [Introduction to Sqitch on PostgreSQL](https://github.com/sqitchers/sqitch/blob/master/lib/sqitchtutorial.pod).
<br/>
<br/>

I've already written about [`sqitch` in the past (in italian)](https://fluca1978.github.io/2014/12/20/calendario-dellavvento-itpug-20-dicembre.html).
<br/>
[`sqitch`](https://sqitch.org/) is a great tool to manage database changes, mainly schema changes. The idea is to provide a `git`-like interface to manage *changes*, a *change* is made by three scripts appropriately written for the backend database:
- a *deploy* script (what to do);
- a *revert* script (how to undo);
- a *test* script (how to check the deploy succeeded).

<br/>
<br/>
### Introducing *sqitchers*.
<br/>
[Around a month ago](https://justatheory.com/2018/05/sqitchers/), the `sqitch` creator, David E. Wheeler, created a GitHub Organization named [`sqitchers`](https://github.com/sqitchers) that now holds all the Sqitch related stuff including, obviously, the codebase for the project. At the same time, the Sqitch steering committee grown, and this is a good thing since this project quickly became an handy tool for database management.
<br/>
<br/>
In conclusion, `sqitch` is growing and getting more free every day. If you are curious about project history and explaination by its own creator David E. Wheeler, [I suggest you listening to this (old) FLOSS Weekly podcast](https://justatheory.com/2014/09/sqitch-on-floss-weekly/).

