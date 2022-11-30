---
layout: post
title:  "pgagroal: getting run-time configuration"
author: Luca Ferrari
tags:
- pgagroal
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
A new command to interactively get the `pgagroal` runtime configuration.

# pgagroal: getting run-time configuration

[pgagroal](https://github.com/agroal/pgagroal/){:target="_blank"}, the fast connection pooler for PostgreSQL, is gaining new features!
In [this commit](https://github.com/agroal/pgagroal/commit/07b79ccd95c2fd709594ea8002c2ea89715adb20){:target="_blank"} I introduced a new *command* for `pgagroal-cli`: **`config-get`**. Such command allows the user to specify the name of a configuration parameter and get back the value the pooler is using.
As an example:

<br/>
<br/>

``` shell
% pgagroal-cli get-config max_connections
300

% pgagroal-cli get-config max_connections --verbose
max_connections = 300
```
<br/>
<br/>

When the command is invoked with the `--verbose` flag, the application respond with a full configuration line that can then be copied and pasted into a new configuration file.


## INI sections

The `config-get` command allows also for the specification of *sections*, for example if you `pgagroal.conf` configuration file is like the following:

<br/>
<br/>

``` shell
% cat pgagroal.conf
[pgagroal]
...

[venkman]
host = 192.168.2.2
port = 5432
primary = off

```
<br/>
<br/>

it is possible to query the command with the section name, and the application will dig into the INI file section:

<br/>
<br/>

``` shell
% pgagroal-cli config-get venkman.host
192.168.2.2

```
<br/>
<br/>

Thanks to this, it is possible to get all the main configuration out of `pgagroal-cli`.

## Limit and HBA entries

Similarly to the sections, the `config-get` command allows the specification of parameters to search for a limit entry or an HBA entry. In such case, the *key* to search for is the username to match for an HBA entry and the database to match for a limit entry. The special key prefix `limit` or `hba` allows the command to understand where to dig.
As an example:


<br/>
<br/>

``` shell
% pgagroal-cli config-get limit.testdb.max_size
10

% pgagroal-cli config-get hba.luca.method
md5

```
<br/>
<br/>

See the [`pgagroal-cli`](https://github.com/agroal/pgagroal/blob/master/doc/CLI.md){:target="_blank"} documentation for more examples and details.
```

```

```

```

```

```

```
