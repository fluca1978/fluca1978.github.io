---
layout: post
title:  "pgenv `config migrate`"
author: Luca Ferrari
tags:
- postgresql
- pgenv
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
`pgenv` 1.2.1 introduces a different configuration setup.

# `pgenv config migrate`

Just a few hours I blogged about some new cool features in [`pgenv`](https://github.com/theory/pgenv){:target="_blank"}, I completed the work about *configuration in one place*.
<br/>
Now `pgenv` will keep all configuration files into a single directory, named `config` . This is useful because it allows you to backup and/or migrate all the configuration from one machine to another easily.
<br/>
But it's not all: since the configuration is now under a single directory, the single configuration file name has changed. Before this release, a configuration file was named like `.pgenv.PGVERSION.conf`, with the `.pgenv` prefix that both made the file hidden and stated to which application such file belongs to. Since the configuration files are now into a subdirectory, the prefix has been dropped, so that every configuration file is now simply named as `PGVERSION.conf`, like for example `10.4.conf`.
<br/>
And since we like to make things easy, there is a `config migrate` command that helps you move your existing configuration from the *old* naming scheme to the *new* one:


<br/>
<br/>
```sh
% pgenv config migrate
Migrated 3 configuration file(s) from previous versions (0 not migrated)
Your configuration file(s) are now into [~/git/misc/PostgreSQL/pgenv/config]
```
<br/>
<br/>

Let's have fun with `pgenv`!
