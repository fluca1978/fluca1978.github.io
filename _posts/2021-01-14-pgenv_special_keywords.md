---
layout: post
title:  "pgenv special keywords: earliest and latest"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A nice addition to the `pgenv` PostgreSQL binary manager.

pgenv special keywords: earliest and latest
---

I recently added support for two different keywords in [`pgenv`](https://github.com/theory/pgenv){:target="_blank"}: **earliest** and **latest**.
<br/>
The idea is quite simple: instead of having to specify each time a PostgreSQL version number to work on, you can now specify one of the above keywords to *jump* immediately to the oldest or newest PostgreSQL version you have installed. Of course, the newest PostgreSQL version is the most recent on a version number basis (not installation time), and on the other hand the oldest is the one with the lesser version number among those installed.
<br/>
Let's understand the concept with an example:


<br/>
<br/>
```shell
% pgenv versions
      12.1      pgsql-12.1
      12.3      pgsql-12.3
      12.4      pgsql-12.4
      13.0      pgsql-13.0
      9.6.20    pgsql-9.6.20
```
<br/>
<br/>


Among the versions installed above, we have that:
- `9.6.20` is the oldest one, and therefore is *mapped* to `earliest`;
- `13.0` is the newest one, and therefore is *mapped* to `newest`.
It is quite easy to demonstrate this by means of `use`:


<br/>
<br/>
```shell
% pgenv use earliest

PostgreSQL 9.6.20 started
Logging to /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log

```
<br/>
<br/>

As you can see, `earliest` has been resolved to version `9.6.20`; on the other hand `latest` is going to be resolved to `13.0`:

<br/>
<br/>
```shell
% pgenv use latest

PostgreSQL 9.6.20 stopped
PostgreSQL 13.0 started
Logging to /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log
```
<br/>
<br/>


But that is not enough: you can also narrow down the scope of versions to a specific major number. For instance, in the `12` branch we have installed `12.1`, `12.3` and `12.4`, that means that `12.1` is oldest version in the twelve branch, as far as `12.4` is the newest one. You can filter by a version number specifying the major version number after the `earliest` or `latest` keywords:


<br/>
<br/>
```shell
% pgenv use latest 12

PostgreSQL 13.0 stopped
PostgreSQL 12.4 started
Logging to /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log


% pgenv use earliest 12

PostgreSQL 12.4 stopped
PostgreSQL 12.1 started
Logging to /home/luca/git/misc/PostgreSQL/pgenv/pgsql/data/server.log
```
<br/>
<br/>


Thanks to the addition of `earliest` and `latest` it becomes more intuitive and easy to automate `pgenv` usage, so that you don't have to remember to which version of PostgreSQL you are referring to.


# What about `build`?
Thanks to [this commit](https://github.com/theory/pgenv/commit/95236fd43f8f7af5f1b94e3fe9259397fcb70c46){:target="_blank"}, **it is now possible to issue a `build` command using the same special keywords as above**.
<br/>
As an example, specifying `pgenv build latest 13` will install the latest available version in the `13` major release, as well as `pgenv build latest` will install the very last available version among all.
The word `earliest` works the opposite, even if I believe that building the very oldest PostgreSQL version could be a good way to have fun!
