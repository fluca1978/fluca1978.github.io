---
layout: post
title:  "Docker Compose V2 (get rid of 'chunked' error)"
author: Luca Ferrari
tags:
- docker
- linux
permalink: /:year/:month/:day/:title.html
---
An error due to the usage of an old version of compose.

# Docker Compose V2 (get rid of 'chunked' error)

Today I started a new container on an Ubuntu machine of mines, and I got the error `TypeError: HTTPConnection.request() got an unexpected keyword argument 'chunked'`.

What the hell is that?

After a little digging, I found that Docker has moved the `compose` version to a fresh `v2` version.
Installing the package `docker-compose-v2` does not suffice, there is the need to remove the old package, so:

<br/>
<br/>
```shell
% sudo apt remove docker-compose
% sudo apt install docker-compose-v2
```
<br/>
<br/>

And after that, all runs smooth as usual. As far as I understand, this new `docker-compose` mimics the one already installed on my Rocky Linux machines.
