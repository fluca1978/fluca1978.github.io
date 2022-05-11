---
layout: post
title:  "pgenv `switch`"
author: Luca Ferrari
tags:
- postgresql
- pgenv
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
`pgenv` 1.3.0 adds a new command: `switch`

# `pgenv `switch`

[`pgenv`](https://github.com/theory/pgenv){:target="_blank"}, a simple but great shell script that helps managing several PostgreSQL instances on your machine, have been improved in the last days.
<br/>
<br/>
Thanks to the contribution of **Nils Dijk** [`@thanodnl` on GitHub](https://github.com/thanodnl){:target="_blank"}, there is now a new command named `switch` that allows you to quickly prepare the whole environment for a different PostgreSQL version without having to start it.

<br/>
The problem, as described in [this pull request](https://github.com/theory/pgenv/pull/53){:target="_blank"} was that the `use` command, trying to be *smart*, starts a PostgreSQL instance once it has been chosen. On the other hand, `switch`, allows you to pre-select the PostgreSQL instance to use without starting it. This is handy, for example, when you want to compile some code against a particular version of PostgreSQL (managed by `pgenv`) but don't want to waste your computer resources starting up PostgreSQL.
<br/>
To some extent, `switch` can be thought as an efficient equivalent of:

<br/>
<br/>

``` sh
% pgenv use 14.2
% pgenv stop
```
<br/>
<br/>

The command has been implemented as a *subcase* of `use`, but while `use` does fire up an instance, `switch` does not.
<br/>
However, in the case an instance is already running, **switching to a new instance will stop the previously running one**!

## Other minor contributions

If you have `pgenv` on the radar, you probably have seen another release in the last days, that covered a bug fix spot by Nils Dijk about the management of the configuration.


# Conclusions

`pgenv` keeps growing and adding new features, and is becoming a more complex beast than it was in the beginning.
Hopefully, it can help your workflow too!
