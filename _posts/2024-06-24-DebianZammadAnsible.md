---
layout: post
title:  "Automating the installation of Zammad on Debian via Ansible: nightmares and scary things!"
author: Luca Ferrari
tags:
- ansible
- zammad
- debian
permalink: /:year/:month/:day/:title.html
---
How hard could it be to work with Debian packages? It turned out it is quite hard!

# Automating the installation of Zammad on Debian via Ansible: nightmares and scary things!

Quite frankly, Debian is not a platform I like very much, and hence it is not a platform I'm used to.

However, sometimes, I have to cope with it.

In particular, I had a rich set of problems when automating the [Zammad Helpdesk](https://zammad.org){:target="_blank"} on a Debian 12 system via Ansible.

First of all, Zammad on Debian by defaults wants a PostgreSQL database, and this is a good thing according to me!
However, if you remove the `zammad` package and try to install it over, everything seems to work until you remove the `zammad` database from your PostgreSQL cluster. You will start **getting errors about the creation of the database**!

And moreover, the *database installation instructions* (i.e., `/opt/zammad/bin/rake db:create`) will fail too!

So I kindly asked for help on [Zammad Support Forum](https://community.zammad.org/t/how-to-create-postgresql-database/14695/3){:target="_blank"} without any luck at all.

At the end, it turned out, that I had to remove the Debian package with the `--purge` option and I have to remove also `/opt/zammad` to let the system reinitialize the database from scratch.

All the steps are documented on my own reply to the [Zammad Support Forum Query](https://community.zammad.org/t/how-to-create-postgresql-database/14695/3){:target="_blank"}.
