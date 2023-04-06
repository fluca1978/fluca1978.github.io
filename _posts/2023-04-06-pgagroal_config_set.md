---
layout: post
title:  "pgagroal: setting configuration at run-time"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
A new feature added to `pgagroal` that allows users to dynamically change the configuration.

# pgagroal: setting configuration at run-time

I'm happy since today my contribution to [pgagroal](https://github.com/agroal/pgagroal/){:target="_blank"} has been merged.
[The last year](https://github.com/agroal/pgagroal/commit/07b79ccd95c2fd709594ea8002c2ea89715adb20){:target="_blank"} I added the `config-get` command to `pgagroal-cli`: such command allowed users to get information about how `pgagroal` was configured.

The natural improvement over the above work would have been the **`config-set`** command, and now `pgagroal-cli` has one (see [this commit](https://github.com/agroal/pgagroal/commit/29902144832662598ebcc324f627627c1595a319){:target="_blank"} !
It took me a few months to complete the work, since I was very busy on my day job: I had a working prototype working before Christmas, but then I let it there for *the future me* to have some time to complete the effort. And in the last month, I had some spare time, so I completed it!

## pgagroal-cli config-set

The new command allows to dynamically change some configuration values. Clearly, not everything can be changed at run-time without a daemon restart. I wanted the command to be useful also in automating scripts, so I thought it could be useful for the command to report back the actual value of a configuration parameter. Therefore, checking the *desiring* value and the *obtained* value can confirm if the change has been applied or not.

Similarly to the `config-get` command, also `config-set` accepts *contexts*:
- `pgagroal` (or nothing) means that the specified configuration parameter is within the `[pgagroal]` configuration section;
- `limit` means that the user requested to change a limit entry;
- `server` the user wants to change something about a server section;
- `hba` the user wants to change an HBA entry.

In the case of `limit`, `hba` and `server` contexts, the entry to modify must be identified with a name. While in a server configuration the name must be unique, the limits and hbas could not have unique names, so the first match wins.

As an example, imagine the user wants to change the `max_connection` setting. Such setting is within the `[pgagroal]` section, so the following are two identical commands:

<br/>
<br/>
```shell
$ pgagroal-cli config-set max_connections 100
40

$ pgagroal-cli config-set pgagroal.max_connections 100
40

```
<br/>
<br/>

In both the above cases, the system is returning `40` instead of the desired value `100`. That's normal, since the `max_connections` requires a restart of the daemon, and can be better understood using the `--verbose` flag:

<br/>
<br/>
```shell
$ pgagroal-cli config-set max_connections 100 --verbose
max_connections = 40
pgagroal-cli: Error (2)
```
<br/>
<br/>

Clearly the system reports there was an error, and provides the information that `max_connections` is set at `40`.


Another example could be when the user decides to change a server or limit value, where he has to specify the context:

<br/>
<br/>
```shell
$ pgagroal-cli config-set server.venkman.port 6432
6432

$ pgagroal config-set limit.pgbench.max_size 2
2
```
<br/>
<br/>

In the above, the user changes the `venkman` server port, and the `pgbench` user limit entry.

If the system, that is the `pgagroal` daemon, cannot apply the requested change on the fly, the logs will be populated accordingly:

<br/>
<br/>
```shell
DEBUG Trying to change main configuration setting <max_connections> to <100>
INFO  Restart required for max_connections - Existing 40 New 100
WARN  1 settings cannot be applied
DEBUG pgagroal_management_write_config_set: unable to apply changes to <max_connections> -> <100>
```
<br/>
<br/>


Clearly the messages may vary depending on the log level configuration.

For more information, please see [the official documentation](https://github.com/agroal/pgagroal/blob/master/doc/CLI.md){:target="_blank"} .
