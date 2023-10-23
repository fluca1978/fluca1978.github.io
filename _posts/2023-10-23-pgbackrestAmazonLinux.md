---
layout: post
title:  "Installing pgBackRest on Amazon Linux (by sources)"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgbackrest
permalink: /:year/:month/:day/:title.html
---
A recap on how to comile pgBackRest on Amazon Linux.

# Installing pgBackRest on Amazon Linux (by sources)

I had the need to install [pgBackRest](https://pgbackrest.org/){:target="_blank"} on Amazon Linux machines.

Unluckily, even if [Amazon Linux 2023](https://aws.amazon.com/linux/amazon-linux-2023){:target="_blank"} is a *Red-Hat like* operating system, the official *PGDG* repository did not install in any version. Therefore, I decided to install from sources, compiling the latest `2.48` version.

In order to achieve the final result, I had to install the following packages:

<br/>
<br/>
```sh
$ sudo dnf install postgresql15-server-devel.x86_64
$ sudo dnf install libxml2-static.x86_64
$ sudo dnf install -y libxml2-devel.x86_64
$ sudo dnf install -y libyaml-devel.x86_64
$ sudo dnf install -y bzip2-devel.x86_64
```
<br/>
<br/>

After this, I was able to download and compile `pgbackRest`:

<br/>
<br/>
```shell
$ wget https://github.com/pgbackrest/pgbackrest/archive/refs/tags/release/2.48.tar.gz
$ tar xzf 2.48.tar.gz
$ cd pgbackrest-2.48/src
$ ./configure && make && sudo make install
```
<br/>
<br/>


I tested it, and it works as solid as only `pgBackRest` can be!
