---
layout: post
title:  "Systemd: service command not found"
author: Luca Ferrari
tags:
- linux
- systemd
- rant

permalink: /:year/:month/:day/:title.html
---
Today I was installing a new service on a Fedora box and I ran into this strange problem.

# Systemd: service command not found

What can you do when the system answers to your commands with something like

```shell
% sudo service postgresql stop
zsh: command not found: service
```

That happened to me today.
<br/>
At glance, I thought there was some hidden problem with the `PATH` and `sudo(1)`, but a quick search on the web shown me that, for some strange reasons, **Fedora has a specific package named `initscripts` that includes `service`**, so the command what effectively not installed on the system.
<br/>
<br/>
The solution was therefore as simple as:

```shell
% sudo yum install initscripts
```

but **why is a such important component not available on the base install?**
<br/>
I know that it is possible to interact with `systemctl` in a straight manner, so for instance the following are equivalent:

```shell
% sudo systemctl stop postgresql
% sudo service postgresql stop
```

but having to manage a lot of different system, I tend to rely on `service` be there and do the right thing depending on the service manager the operating system is using.
