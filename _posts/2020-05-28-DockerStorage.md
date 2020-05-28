---
layout: post
title:  "Docker & Systemd: changing the storage path"
author: Luca Ferrari
tags:
- docker
- linux
- systemd
permalink: /:year/:month/:day/:title.html
---
First tiny steps into the docker world.

# Docker & Systemd: changing the storage path

When using Docker on a Systemd based system, like CentOS, the best way to change the storage path used to download images is by means of `systemctl`.
<br/>
For example, I did the following:

```shell
$ sudo systemctl edit docker
```

that pops up your default editor, then you need to insert the following lines:


```shell
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -g /your/storage/path
```

It is important to empty the `ExecStart` property first, and then add the storage path with the `-g` option.
The other options are the same you can see in the main docker file.
<br/>
Once your configuration is ok, ensure that systemd applied it (and the *Drop-In* line is there):

```shell
$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─override.conf
   Active: active (running) since mer 2020-05-27 16:31:43 CEST; 19h ago
     Docs: https://docs.docker.com
 Main PID: 24099 (dockerd)
    Tasks: 27
    Memory: 53.8M
```
