---
layout: post
title:  "pgenv get configuration!"
author: Luca Ferrari
tags:
- postgresql
- itpug
- planet-postgresql-org
- pgenv
permalink: /:year/:month/:day/:title.html
---
I have already written about a very useful and powerful small pearl by   [*theory*](https://justatheory.com/): [`pgenv`](https://github.com/theory/pgenv). Now the tool does support for user configuration!

# pgenv get configuration!
I spent some time implementing a very rudimental approach to configuration for `pgenv`.
The idea was simple: since the program is a single Bash script, the configuration can be done using a single file to source variables in.

But before this was possible, I had to do a little refactoring over here and there in order to make all the commands behave smooth across the configuration. And at least, it seems to work, with some parts that can be improved and implemented better (as always it is!). However, I designed from scratch to support every single version of PostgreSQL, that means configuration could be different depending on the specific version you are running. This allows, for example, to set particular flags for ancient versions, without having to get crazy when switching to more recent ones.


Now `pgenv` supports a `config` command that, in turn, support for several subcommands:
- `write` to store the configuration in an hidden file named after the PostgreSQL version (e.g., `.pgenv.10.5.conf`);
- `edit` just to launch you `$EDITOR` to manipulate the configuration;
- `delete` to remove a configuration file;
- `show` to dump the configuration.

The idea is simple: each time a new PostgreSQL version is built, a configuration file is created for such instance. You can then customize the file in order to make `pgenv` behave differently for that particular version of PostgreSQL. As an example, you can set different languages (e.g., PL/Perl) or different startup/stop modes.
If the configuration file for a particular version is not found, a *global* configuration is loaded. If neither that is found, the program behaves depending on some shell variables or, as last resort, with its internal defaults.

Please read the [documentation](https://github.com/theory/pgenv/blob/master/README.md) for a better explaination of the new `config` command.
