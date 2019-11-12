---
layout: post
title:  "PostgreSQL ascii logo for FreeBSD boot loader"
author: Luca Ferrari
tags:
- freebsd
- postgresql
- planet-postgresql-org


permalink: /:year/:month/:day/:title.html
---
I spent some time making an elephant logo to be used as FreeBSD boot loader logo.

# PostgreSQL ascii logo for FreeBSD boot loader

I use FreeBSD as my main PostgreSQL server, and also as virtual machine for training courses. A long time ago, I changed the *message of the day* (`/etc/motd`) to reflect the elephant logo in ascii-art, but why not changing also the booloader logo?
<br/>
FreeBSD by default shows what is called *orb* or the devil (named *beastie*), and the new [Lua](https://www.lua.org/) based bootloader use some *simple string concatenation* to generate a logo.
However, it was not so simple to make a new logo, since I've no idea about how to debug it production, and that forced me to a very long and repetitive *try and reboot** process to identify all the problems with my logos.
<br/>
**Last, I made it!** 
<br/>
Now there are two available logos for the bootloader that provide both the black-and-white and the coloured elephant. Below you can see a couple of screenshoots:

<br/>
<br/>
<center>
<img src="/images/posts/freebsd_logo/logo-postgresql-color.png" />
<br/>
<br/>
<img src="/images/posts/freebsd_logo/logo-postgresql-bw.png" />
</center>
<br/>
## How to use the PostgreSQL bootloader logo

In order to use one of the logos, you have to:
- download the Lua script from my [Github repository](https://github.com/fluca1978/fluca1978-pg-utils/tree/master/logos), within the `logos` directory you can find the files
   - [`logo-postgresql.lua`](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/logos/logo-postgresql.lua) that is the coloured version of the logo;
   - [`logo-postgresqlbw.lua`](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/logos/logo-postgresqlbw.lua) that is the black-and-white version of the logo;
- put the choosen file into the `/boot/lua` directory and provide read permissions;
- edit your `/boot/loader.conf` and add the setting `loader_logo` depending on the chosen logo
```shell
# for the coloured version
loader_logo="postgresql"
# or for the black-and-white version
# loader_logo="postgresqlbw"
```
- and of course, reboot!

### Why `cyan` instead of `blue`?

You probably have noticed that the coloured elephant is made in `cyan` and not in the well known `blue`. The reason for that is that the console foreground blue is too dark to make the elephant appear. However, it is possible to manipulate the escape sequences in order to get a different color, but please note that for a reason I don't know, highlighting colors (e.g., escape sequences like `94`) are not working in the bootloader.


## How to use the `/etc/motd` logo

In the [logos directory](https://github.com/fluca1978/fluca1978-pg-utils/tree/master/logos) there is also the [motd](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/logos/motd) file, or better, an example of message-of-the-day. Place it on your machine and customize at your wills.

## What about the ascii art?

Credits to the ascii art go to Oleg Bartunov, even if I'm not able to find out anymore a message thread where he proposed the elephant logo. However, thanks also to [Charles Clavadetscher](https://www.postgresql.org/message-id/57386570.8090703%40swisspug.org) that provided another version.

## Feel free to contribute!

As usual, having posted the logos on my [Github repository](https://github.com/fluca1978/fluca1978-pg-utils/tree/master/logos), any contribution and improvement is welcome.
