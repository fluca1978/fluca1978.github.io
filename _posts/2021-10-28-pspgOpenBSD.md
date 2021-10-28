---
layout: post
title:  "pspg lands in OpenBSD" 
author: Luca Ferrari
tags:
- openbsd
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A great pager into a great operating system.

# pspg lands in OpenBSD

[`pspg`](https://github.com/okbob/pspg{:target="_blank"}) is a great pager specifically designed for PostgreSQL, or better, for `psql`, the default and powerful text client for PostgreSQL databases.
<br/>
But `pspg` is more than simply a *pager for PostgreSQL*: it is a general purpose pager for tabular data.
<br/>
<br/>
It happened that a few weeks ago I was using an OpenBSD system, and since I had to do some work with PostgreSQL, I decided to install `pspg` to get some advantages. Unluckily, there was no package for OpenBSD, and most notably, no port in the ports tree.
<br/>
Therefore, the only chance to install `pspg` was to compile it from sources, but I failed. I [opened an issue](https://github.com/okbob/pspg/issues/189){:target="_blank"} to get some help, and after some assistance, I decided to dig deeper. So [I asked for help on the `misc` OpenBSD mailing list](https://marc.info/?l=openbsd-misc&m=163402630004343&w=2){:target="_blank"} and get much more that I was expecting: not only I solved the problem on how to install `pspg`, but the application was noticed and a proposed for a new port was issued.
<br/>
In fact, another italian guy, Omar, [did prepared and proposed a `pspg` port](https://marc.info/?l=openbsd-ports&m=163404042013173&w=2){:target="_blank"}, and after a few days the [port get included into the ports tree](https://cvsweb.openbsd.org/cgi-bin/cvsweb/ports/databases/pspg/){:target="_blank"}!
<br/>
<br/>
What does tha mean? That, at least at the moment of writing, that you can get `pspg` installed on OpenBSD via the ports:


<br/>
<br/>
```shell
% cd /usr/ports/databases/pspg
% doas make install  
===> pspg-5.4.0 depends on: postgresql-client-* -> postgresql-client-13.4p0
===> pspg-5.4.0 depends on: readline-* -> readline-7.0p0
===> pspg-5.4.0 depends on: metaauto-* -> metaauto-1.0p4
===> pspg-5.4.0 depends on: autoconf-2.69 -> autoconf-2.69p3
===> pspg-5.4.0 depends on: gmake-* -> gmake-4.3
===>  Verifying specs: c curses ereadline m panel pq
===>  found c.96.1 curses.14.0 ereadline.2.0 m.10.1 panel.6.0 pq.6.12
===>  Installing pspg-5.4.0 from /usr/ports/packages/amd64/all/
pspg-5.4.0: ok

```
<br/>
<br/>

It is important to note that the ports tree that include `pspg`, at the time of writing, is the `-CURRENT` (see [here](https://www.openbsd.org/faq/ports/ports.html){:target="_blank"}), and therefore there is still some time to wait to get `pspg` as a package and a port in the `-RELEASE` ports tree.

## Great OpenBSD Job!

I must say that I was astonished by the great work done by [Omar](https://www.omarpolo.com/){:target="_blank"} and the other OpenBSD volunteers to get the `pspg` within the ports tree. 


# Conclusions

`pspg` is a very useful and interesting pager for tabular like data, and of course this includes output from PostgreSQL's `psql` command line client.
<br/>
With a bit of luck, patience, and the effort of the OpenBSD community, this program will be soon available on OpenBSD too as a package!

