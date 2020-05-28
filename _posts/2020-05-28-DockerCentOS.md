---
layout: post
title:  "Docker & CentOS: docker-proxy is not in your PATH"
author: Luca Ferrari
tags:
- docker
- linux
- centos
permalink: /:year/:month/:day/:title.html
---
First tiny steps into the docker world.

# Docker & CentOS: docker-proxy is not in your PATH

I'm not a big fan of Docker, coming from a Unix background I don't see the hype and I'm quite happy with *FreeBSD jails*, that as far as I know are really similar to the *container thing*** Docker is about.
<br/>
Anyway, a few days ago I was fighting with a Docker installation on a CentOS server.
<br/>
It turned out it was not so simple, and in fact **installing Docker from the distro repositories is a very bad idea!**
<br/>
While *dockering* around, I came across a strange error about `docker-proxy is not in your PATH`. The reason is that CentOS does not install "plain" `docker` executables, rather something like `docker-current` and `docker-proxy-current`. Searching the web revealed that a simple symbolic link or a rename could fix the problem, and so it was. Or it seemed to be.
<br/>
But while all the docker-stuff was working, I came again across the very same issue during a build.
<bR/>
At that point I decided to nuke the CentOS Docker packages and to install them *the official way*, so [following the official Docker instructions for CentOS](https://docs.docker.com/engine/install/centos/){:target="_blank"}, I did:

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
$ sudo yum install docker-ce docker-ce-cli containerd.io    
```

and the build was able to complete!
<br/>
<br/>
**Lesson learned: *when dealing with a tool that I don't know well, on an operating system I'm not used too, it is better to follow the official installation instructions*. Even if they are longer than a package manager `install` command!**
