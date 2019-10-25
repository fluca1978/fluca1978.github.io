---
layout: post
title:  "pgenv: adjust your PATH!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A few days ago we added the option to suggest you changes to your `PATH` to prevent version clashes.

# pgenv: adjust your PATH!

In the following you can find another quick video that demonstrate how easy it is to get, almost *automtically*, a PostgreSQL 12 instance up and running on your local machine using [`pgenv`](https://github.com/theory/pgenv).

[![asciicast](https://asciinema.org/a/276923.svg)](https://asciinema.org/a/276923)

Please note also that, at time `5:35`, you will see how `pgenv` suggests you to adjust your `PATH` environment variable in order to use the *just installed* binaries for the cluster. The idea behind this suggestion is to prevent you using a system-wide binary, e.g., `psql`, that has a possible incompatibility with the *in-use* cluster.
