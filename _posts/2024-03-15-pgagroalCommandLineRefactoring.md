---
layout: post
title:  "pgagroal command refactoring (again!) and a new contributor!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
Changes in `pgagroal-cli` and `pgagroal-admin`.

# pgagroal command refactoring (again!) and a new contributor!

Last year I introduced a way in `pgagroal-cli` and `pgagroal-admin` to arrange commands in a more consistent and manageable way, deprecating some commands too.

Today, a new contributor to the project, **Henrique de Carvalho**, [committed a patch](https://github.com/agroal/pgagroal/commit/11a1f475631534797aab63d6ad34d3990f0f9e70){:target="_blank"} that greatly improves the way commands are handled internally.

The users will not notice any particular difference, except that also [a bug has been fixed in handling deprecated commands](https://github.com/agroal/pgagroal/commit/d9f9253504605194b6bac1dd18491f132059b145){:target="_blank"}, but the changes in the code are very important: now all the commands are organized in a list of `struct`s that provide a more accurate way of handling errors, missing arguments or command parts, and logging.

I became thinking about this refactoring months ago, but never got the time to dig into the changes.
However, it all began with an annoying problem with some mispelled commands, that reported a wrong error message to the user.

And now, thanks to the contributions of Henrique, `pgagroal` has done another step towards a more complete and robust system.
