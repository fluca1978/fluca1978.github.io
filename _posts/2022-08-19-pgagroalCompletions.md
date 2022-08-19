---
layout: post
title:  "Shell completions for pgagroal"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
A small patch to ease the use of `pgagroal` tools.

# Shell completions for `pgagroal`

In the beginning of the current month I pushed a commit that introduces shell completions for `pgagroal` related commands, in particular `pgagroal-cli` (used to manage the pooler) and `pgagroal-admin` (used to manage authentication and users).
<br/>
The [shell completions](https://github.com/agroal/pgagroal/commit/1296cc4216c73119a1ff4c3a3ffd0c610ca04f69){:target="_blank"} work only for Bash and Zsh, and allow you to hit `<TAB>` after a command and get it automatically completed with the appropriate options.
<br/>
While importing the completions in Bash is as simple as `source`ing the file, in Zsh you need to enable the completion framework. [Detailed instructions about how to enable the completions](https://github.com/agroal/pgagroal/blob/master/doc/tutorial/01_install.md#shell-completion){:target="_blank} have been placed in the tutorials.
