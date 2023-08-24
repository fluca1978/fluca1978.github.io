---
layout: post
title:  "pgenv gains a new command (and behavior): config path"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv
permalink: /:year/:month/:day/:title.html
---
A minor change to the way `pgenv` handles configuration files.

# pgenv gains a new command (and behavior): config path

I recently merged a minor change to the way [`pgenv`](https://github.com/theory/pgenv){:target="_blank"} handles configuration files.

There is now a new environment variable, named `PGENV_CONFIGURATION_FILE` that can be set outside of the program, i.e., before the `pgenv` is invoked, to point the application to a specific configuration file. Before this change, `pgenv` used to keep a *per-PostgreSQL-version* configuration file, and while this has advantages, required the user to manually merge changes into every configuration file if needed.
<br/>

Now, `pgenv` can exploit the same configuration file for different PostgreSQL versions, assuming `PGENV_CONFIGURATION_FILE` has been exported to the value of such file:

<br/>
<br/>
```shell
% export PGENV_CONFIGURATION_FILE=/home/luca/my-penv.conf
% pgenv build 16beta2
...
% pgenv build 15.0
```
<br/>
<br/>

In the above example, both the `build` commands will use the same configuration file pointed by the `PGENV_CONFIGURATION_FILE`` environment variable.

<br/>

But there's more!
<br/>
Now `pgenv` refuses to write a new configuration file for a specific version if the `PGENV_CONFIGURATION_FILE` environment variable is set.
This is a change with the previous behavior of `pgenv`, where whnever possible, the application wrote a new (cloned) configuration file. This does mean you cannot use `config write` to write a new configuration file, but `pgenv` will avoid creating a new file implicitly.

<br/>
Having granted much more value to the configuration file path, how can you know where it is?
<br/>
`pgenv` now supports a new command `config path` that will show you the configuration path for every version you are going to use.
For example, imagine you want to replicate your PostgreSQL 15 `pgenv` configuration against the new version 16, you can do:

<br/>
<br/>
```shell
% export PGENV_CONFIGURATION_FILE=$( pgenv config path 15.0 )
% pgenv build 16beta2

```
<br/>
<br/>
