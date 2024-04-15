---
layout: post
title:  "pgenv: run once scripts"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A new feature to run a single script at the very beginning of the cluster lifecycle.

# pgenv: run once scripts

Today [pgenv](https://github.com/theory/pgenv){:target="_blank"} got a new release that provides a simple, but quite useful, feature: **the capability to run a custom script the first time the instance is started**.

The idea is simple: after the `initdb` phase, if the user has configured a `PGENV_SCRIPT_FIRSTSTART` executable, the system will run such script against the (just) started instance. This is different from `PGENVE_SCRIPT_POSTSTART` script, since the latter ie executed *every time the cluster has started*, while `PGENV_SCRIPT_FIRSTSTART` is run only the first time the database cluster is started.

The aim of this script is, hence, to install users and databases, or populate some initial data.
