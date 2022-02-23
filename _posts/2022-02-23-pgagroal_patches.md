---
layout: post
title:  "Contributing to pgagroal (and pgmoneta?)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
- pgmoneta
permalink: /:year/:month/:day/:title.html
---
My small contributions to two interesting projects.

# Contributing to `pgagroal` (and `pgmoneta`?)

`[pgagroal](https://agroal.github.io/pgagroal/){:target="_blank"}` is an interesting *high-performance* PostgreSQL connection pooler. I started using and studying it at the end of the past year, and due to some discussions on the [project discussions page](https://github.com/agroal/pgagroal/discussions){:target="_blank"}, I decided to have a look at the source code.
<br/>
The above resulted in a few small contributions that have been merged:
- [small changes to configuration error messages](https://github.com/agroal/pgagroal/pull/202){:target="_blank"}. During some testing I noted that there was a misleading, at least to me, `FATAL` log message when the configuration of *limits* was inconsistent; this patch tries to improve the situation that was not strictly related to the log message, rather to the way of throwing the configuration inconsistency thru the application stack. The patch has been squashed and merged.
- [master key error messages](https://github.com/agroal/pgagroal/pull/207){:target="_blank"} I was frustated one day while trying to insert a *master-key* password for the `pgagroal` *vault*, but since I was inserting a too short password, the system kept asking me the password without any error message. Thsi patch, squashed and merged, improves the verbosity of the application in such condition.
- [generate a PID file depending on the listening socket](https://github.com/agroal/pgagroal/pull/201){:target="_blank"} was required because I was not able to stop `pgagroal` sometime. The problem was that launching two different instances on the same machine could result in some problems when the configuration was duplicated. The solution was to use a *guard* PID file based on the listening socket, so that the daemon could not be started on the same host in such conditions.
- [verbosity about master-key vault](https://github.com/agroal/pgagroal/pull/209){:target="_blank"} provides an informational string to the user about *where*, on disk, the vault has been stored.

<br/>
<br/>
The above are very small contributions, due also to the fact that I don't know (yet) `pgagroal` so well to be confident in doing more complex contributions. Also, as you can see from the pull requests, my Emacs decided to wipe out several times the code style, resulting in merge to stay pending. Moreover, it has been years since I developed something in C!

<br/>
<br/>
Getting in touch with `pgagroal` lead me to get to know also another interesting project: `[pgmoneta](https://github.com/pgmoneta/pgmoneta){:target="_blank"}`, a backup solution for PostgreSQL.
I did not have very much time to inspect and study this project, but at least I was able to re-propose a similar patch about [the guard PID file](https://github.com/pgmoneta/pgmoneta/pull/43){:target="_blank"}.

<br/>
<BR/>
What's next? Well, I'm trying to implement *log rotation* on `pgagroal`, but I'm not yet ready to push in the wild a proposal. So far, I'm testing my work, so stay tuned!
