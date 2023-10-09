---
layout: post
title:  "Docker (and docker-compose) on Rocky Linux"
author: Luca Ferrari
tags:
- linux
- docker
permalink: /:year/:month/:day/:title.html
---
It is not as simple as you might think to get Docker up and running on Rocky Linux.

# Docker (and docker-compose) on Rocky Linux

I am not a great fan of Docker, but I have to use somehow for several activities.
So far, I've always used Docker on Ubuntu based distributions, were installing a decent `docker` and `docker-compose` version was quite simple and did not require any particular repository to set up.

This is not the case with Rocky Linux.

First of all, to get `docker` running on Rocky Linux there is the need to install a particolar repository:

<br/>
<br/>
```shell
% sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
<br/>
<br/>

Then, you can install the packages `docker-ce` and `docker-ce-cli` to get almost all the commands and machinery up and running. In particular, you have to:

<br/>
<br/>
```shell
sudo dnf -y install docker-ce docker-ce-cli
```
<br/>
<br/>

However, the above is not going to give you the `docker-compose`, and you need also to install the following:


<br/>
<br/>
```shell
% sudo dnf install docker-compose-plugin
```
<br/>
<br/>

But, unluckily, this is not going to give you the `docker-compose` command again: on Rocky Linux `docker-compose` does not exist and `compose` is a subcommand of `docker`, that means it has to be invoked as **`docker compose`**.

This is clearly a problem for scripts using `docker-compose` that need to be adjusted to use `docker compose` instead, and I don't know the reasons for this **non-portable** choice!


Last, in order to run `docker` you have to start the service:

<br/>
<br/>
```shell
% sudo systemctl --now enable docker
```
<br/>
<br/>



## How to change `docker-compose` to `docker compose`

There are different ways to switch between a version and the other.
The simplest one is to create a shell alias:

<br/>
<br/>
```shell
% alias docker-compose='docker compose'
```
<br/>
<br/>


Another approach is to test, in your script, for the existance of `docker-compose` or `docker compose`:

<br/>
<br/>
```shell
DOCKER_COMPOSE=$(which docker-compose 2>/dev/null)
if [ ! -x "$DOCKER_COMPOSE" ]; then

    # try to run docker compose
    `docker compose > /dev/null 2>&1`
    if [ $? -eq 0 ]; then
		DOCKER_COMPOSE="docker compose"
    fi
fi

```
<br/>
<br/>

In the end, the variable `DOCKER_COMPOSE` will contain the correct command to invoke, so you are free for instance to run `$DOCKER_COMPOSE build` and the script will translate it to either `docker-compose build` or `docker compose build` according to which command has been found.



# Conclusions

Docker is not as portable as it may sound: apparently different distributions use different approaches to some commands.
Personally, I do prefer the Rocky Linux approach (i.e., adding commands to `docker`), but in anycase this is not a portable approach.
