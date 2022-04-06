---
layout: post
title:  "pgbadger incremental mode via SSH"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgbadger
permalink: /:year/:month/:day/:title.html
---
How great it is `pgbadger`?

# pgbadger incremental mode via SSH

[pgbadger](https://github.com/darold/pgbadger){:target="_blank"} is a great tool, and quite frankly I suggest everyone I talk to about PostgreSQL to install it!
<br/>
Why?
<br/>
It is *cheap* and does its job in analyzing logs and providing you insights about what happened in your cluster.
<br/>
<br/>
A few days ago I caught a strange, to me, behavior. `pgbadger` has a very handy *incremental mode* that allows you to keep it running processing new logs every day (or whatever period you choose) and get historical and up-to-date insights.
Well, when downloading a file over an SSH connection, this incremental behavior was not working.
<br/>
Uhm, I was sure it was working, since I use it quite often, but I was unable to understand what I was missing in the configuration of `pgbadger`. After a few experiments and comparisons with other working systems of mines, I found that the `-r` (*remote*) flag was able to work over SSH, while a "simple" URI like `ssh://me@you//var/postgresql/logs` was not.
<br/>
<br/>
[I reported the issue](https://github.com/darold/pgbadger/issues/723){:target="_blank"}, and in less than a week [the problem was fixed](https://github.com/darold/pgbadger/commit/6a4750b35a49ed2f9153315a3642aea3c27db556){:target="_blank"}!
<br/>
Well, this is unfair: it is true that the commits is a week after the initial issue, but **[after only 48 hours there was a commit aimed to fix the problem](https://github.com/darold/pgbadger/issues/723#issuecomment-1078753300){:target="_blank"}**, but then there was some around-the-daylight time spent in communicating tests and their results.
<br/>
<br/>
*This is something you simply don't get in your commercial ecosystem!*
<br/>
<br/>
Thanks for the great work and keep this useful project going!
