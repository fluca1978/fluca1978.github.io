---
layout: post
title:  "pgagroal: where is my configuration?"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
A new command to display where the configuration files are located.

# pgagroal: where is my configuration?

I [implemneted a new command in pgagroal](https://github.com/agroal/pgagroal/commit/5988613a32332c13bc6df8b71290e2989bd711e0){:target="_blank"} `conf ls`. The aim of the command is very simple: display where the configuration files are located.
In fact, `pgagroal` configuration is split into several configuration files, and sometimes it could be useful to get information from the runtime system where a configuration file is.

The command works as follows:

<br/>
<br/>
```sh
% pgagroal conf ls
Main Configuration file:   /etc/pgagroal/pgagroal.conf
HBA file:                  /etc/pgagroal/pgagroal_hba.conf
Limit file:                /etc/pgagroal/pgagroal_databases.conf
Frontend users file:       /etc/pgagroal/pgagroal_frontend_users.conf
Admins file:               /etc/pgagroal/pgagroal_admins.conf
Superuser file:
Users file:                /etc/pgagroal/pgagroal_users.conf
```
<br/>
<br/>

If a configuration file has not been specified, the corresponding value will be left empty, otherwise, the full path to the configuration file will be displayed.

This is another small addition towards a better consistent and useulf command line interface.
