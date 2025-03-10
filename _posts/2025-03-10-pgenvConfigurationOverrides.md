---
layout: post
title:  "pgenv 1.4.0 is out!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgenv
permalink: /:year/:month/:day/:title.html
---
A new version with an interesting improvement in the configuration management.

# pgenv 1.4.0 is out!

[pgenv 1.4.0](https://github.com/theory/pgenv/releases/tag/v1.4.0){:target=_blank} is out with an interesting improvement regarding the configuration management.

When you install, and then use, a specific PostgreSQL version, `pgenv` loads the configuration to start the instance with from a configuration file that is named after the PostgreSQL specific version. For instance, if you are running version `17.1`, then `pgenv` will load the configuration from a file named `17.1.conf`. If the latter file does not exists, the `pgenv` script will try to load the default configuration file `default.conf`.

Now, thanks to the work done in the `pgenv` development, it is possible to allow for multiple configuration files with overrides.
In particular, `pgenv` will load more than one configuration file with **narrowing context** related to the PostgreSQL version.
Therefore, using a `17.1` PostgreSQL version will trigger the loading of the following files:
- `default.conf`
- `17.conf`
- `17.1.conf`

Note the addition of the **major version specific configuration file** (in the above `17.conf`).

This new configuration loading chain will make `pgenv` to load configuration from a default to a specific context, allowing also for a quicker sharing of configuration assuming you are interested only in the major version configuration.
