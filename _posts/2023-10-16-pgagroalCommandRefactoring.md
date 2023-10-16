---
layout: post
title:  "pgagroal command refactoring"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
A new, cleaner, set of commands for `pgagroal`.

# pgagroal command refactoring

It took me more than one year to get [this patch in](https://github.com/agroal/pgagroal/commit/ade40240317bad155dbf1e40866c96257b688b90){:target="_blank"}! The reason was not that this piece of code is particularly complex, rather it is hitting pretty much all the human interface `pgagroal` is exposing to the user.

When I started using [pgagroal](https://github.com/agroal/pgagroal){:target="_blank"} I felt uncomfortably with its command line interface.
Commands had been added as the project improved, but there was not a clear grouping of related commands, and most of them have weird meaning, at least to me.

As an example, the `reset` command was dealing with the Prometheus reset, while `reset-server` was truly dealing with `pgagroal` reset. That sounded weird to me, since I believe `pgagroal` should deal first with itself, and then with other components, so the order of the commands appeared wrong to me.

Another example was the `flush-xxx` set of commands: `flush-gracefully`, `flush-all` and `flush-idle`. Why not grouping those commands into a big `flush` group and then add as a parameter what to effectively flush?

You probably get the point behind my rants. That's why I started to develop [this patch in](https://github.com/agroal/pgagroal/commit/ade40240317bad155dbf1e40866c96257b688b90){:target="_blank"} to refactor the command line of `pgagroal-cli` and `pgagroal-admin` to:
- have command groups
- provide more concise and sane defaults
- handle commands and subcommands, like `git` and other command line oriented tools do.

Since this change would have broke the command line, I decided also to place warnings and accept the *old* commands. Therefore I developed this patch with the option to parse the new set of commands, as well as the old ones, printing out a warning if the user was still using the old ones. This makes retro-compatibility easy, and pushes the user towards the new set of commands to prevent that the future removal of the old commands will break some external tools or scripts.

I'm not going to discuss the whole set of new commands, since the [documentation](https://github.com/agroal/pgagroal/blob/master/doc/CLI.md){:target="_blank"} already does this task. However, just to give you an idea about how the command now looks like, consider the `flsuh` commands:

<br/>
<br/>
```shell
# old commands
$ pgagroal-cli flush-all
$ pgagroal-cli flush-idle
$ pgagroal-cli flush-gracefully

# new commands
$ pgagroal-cli flush all
$ pgagroal-cli flush idle
$ pgagroal-cli flush gracefully

$ pgagroal-cli flush  # same as flush gracefully
```
<br/>
<br/>

As you can see, the new command is just `flush` and it accepts a *subcommand* that can either be `idle`, `all`, `gracefully` and is used to specify the *mode* to execute the `flush` command. This also introduces a new default behavior: if the user does not specify how to flush, the graceful mode is automatically selected.

If the user types in the old command, `pgagroal-cli` will emit a warning like the following:

<br/>
<br/>
```shell
$ pgagroal-cli flush-idle
WARN: command <flush idle> has been deprecated by <flush idle> since version 1.6.0

```
<br/>
<br/>

In this way, we keep compatibility with previous versions while trying to teach the users the new way to execute a command.

**Sooner or later, old commands will be removed!** Therefore users should start updating their tools and scripts to the new interface.


The `conf` set of commands is probably the one that groups the most subcommands. In fact, the `conf` command includes the `set`, the `get` and `reload` commands that were respectively `config-set`, `config-get` and `reload`. Note how the reload subcommand now makes it clearer what is going to be reloaded (i.e., the `conf`).
In other words, according to me, `pgagroal-cli conf reload` is much clearer and less error prone than `pgagroal-cli reload`.

Some commands changed their name, and this was due to a clash in the default actions. For example, the `reset` and `reset-prometheus` commands have been moved into the `clear` group, with `clear server` (the default) and `clear prometheus` respectively.


Similarly, also `pgagroal-admin` has been updated, so that for instance `pgagroal-admin add-user` is now `pgagroal-admin user add`.

Thanks to the very well structured `pgagroal` source code, and to the introduction of a few new utility functions to handle command line arguments, these changes will allow the introduction to new commands and groups with a more consistent command line experience!
