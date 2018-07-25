---
layout: post
title:  "PostgreSQL 11 and PL/Java: it can work!"
author: Luca Ferrari
tags:
- postgresql
- pljava
- itpug
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
PL/Java is a wonderful piece of code that allows the definition of functions in Java directly within PostgreSQL. Unluckily, PostgreSQL 11 introduced a few changes that made PL/Java not compiling. But keep calm, experts are already working on this!

# PostgreSQL 11 and PL/Java: it can work!

## tl;dr

If you are in a rush and want to experiment with PL/Java 1.5.1 Beta and PostgreSQL 11, have a look at this [pull request](https://github.com/tada/pljava/pull/161).

## Trying to compile PL/Java against PG11b2

First of all, I'm used to serve PostgreSQL over FreeBSD, and this is not the optimal situation because a lot of code is written with Linux in mind and requires some adjustements when compiled/ported over other Unix implementations.
PL/Java is an example: it compiles thru Apache Maven, that in turn seems to require GCC, that is not the default compiler on FreeBSD and ... you get the point.

<br/><br/>

However, with a little work, it is possible to compile PL/Java even on FreeBSD (as you can imagine) and this is what I've done so far. But last monday, trying to compile it against PostgreSQL 11 beta 2, quickly resulted in frustation.

<br/>
<br/>

Luckily, and thanks to the great PL/Java, I found that due to a change in the PostgreSQL 11 GUC definition, [things could have been adjusted to make it compiling](https://github.com/tada/pljava/issues/160#issuecomment-407104952). After one day, all of my PL/Java code seemed to be running fine with this simple workaround (and yes, I don't have code that complex, so this does not mean the workaround is production safe!). I therefore decided to open a [pull request about it](https://github.com/tada/pljava/pull/162).
<br/>
<br/>
*Shame on me!*
<br/>
My pull request resulted in a mess, because I made it against the right branch and tag. And in the meantime, I didn't noticed that the [right pull request](https://github.com/tada/pljava/pull/161) was added by someone realy much more competente than me on this subject: Chapman Flack.
Thanks Chapman, hope things will be fixed soon!
