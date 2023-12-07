---
layout: post
title:  "Learn PostgreSQL (second edition): screencasts available!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
An example of how to run the Docker images provided by the Github repository.

# Learn PostgreSQL (second edition): screencasts available!

One of the improvement we made while rewriting and updating the book
was to introduce *Docker images* that the readers can launch as a **safe environment to test the concepts expressed in the book**.
This has several advantages, most notably the fact that the user does not need to install a separate PostgreSQL instance on her own, and fill it with the data that could slightly change from chapter to chapter according to diffent examples. Another advantage is that, in the case the user damages the data and wants to restore it, the container can be erased and a new one can be built from scratch.

In order to help users to quickly access the PostgreSQL containers, preventing them from writing long and boring Docker commands, we built a simple shell script named **`run-pg-docker.sh`** that optionally accepts the name of the chapter image and projects the user within the container logging in as the `postgres` user. Therefore, with just a simple command, the reader can *jump* into the PostgreSQL container and start running all the commands and examples detailed in the book!

And, in order to better demonstrate how to quickly jump in, there are a couple of *asciinema* screencasts to let the readers see how the container process is launched.


The first screencast shows how to run the so called `standalone` container, the *catch-all* container used whenever there is no need for a per-chapter specific container:

<br/>
<br/>
<center>
<a href="https://asciinema.org/a/625735" target="_blank"><img src="https://asciinema.org/a/625735.svg" /></a>
</center>
<br/>
<br/>

The second screencast, on the other hand, shows how to run a specific per-chapter container, in particular the Chapter 10 container (related to users, roles and permissions):


<br/>
<br/>
<center>
<a href="https://asciinema.org/a/625738" target="_blank"><img src="https://asciinema.org/a/625738.svg" /></a>
</center>
<br/>
<br/>


Please consider that the time required for the container to fire up depends on the speed of the Internet connection, of the host machine and on the already downloaded artifacts (i.e., re-launching a container for the second time will require less time).


## Resources

The Github repository for downloading examples, Docker images (via `docker-compose`) and in general source files is available at [at this URL](https://github.com/PacktPublishing/Learn-PostgreSQL-Second-Edition){:target="_blank"}.

The **[Learn PostgreSQL second edition book can be found at this link](https://www.packtpub.com/product/learn-postgresql-second-edition/9781837635641){:target="_blank"}**.

Please consider that some output of the screencasts could be different from the one you get on your system, and that during time the configuration files for the Docker images could slightly change depending on readers' suggestions and comments.
