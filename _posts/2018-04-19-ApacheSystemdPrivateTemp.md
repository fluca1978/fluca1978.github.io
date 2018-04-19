---
layout: post
title:  "Apache, PHP, Systemd and PrivateTemp"
author: Luca Ferrari
tags:
- linux
- php
- development
permalink: /:year/:month/:day/:title.html
---
While `systemd` is a great tool, the way it is (ad)used by Linux distributions could make developers spend a few hours trying to solve stupid and trivial problems... as writing a simple file to `/tmp`!

# PrivateTemp when running PHP and Apache

On my development machine I've got a PHP application that writes out several stuff on `/tmp`. The choice of such directory, as not-smart as it is, allows the application to able to write out data without having me to spend time adjusting permissions and a full logging system. Surely, it could be better, however it does its job.

So far so good, a few days ago I fired up my application from the command line, and it worked as expected.
Then I ran again from the browser, i.e., from within the web server, and it was not writing nothing anymore. It seemd it was executing fine, but no messages and no update was issued on the *log* files.




In the beginning I suspected a permission problem, but quite early I discovered it was not such a problem. I then tried to dig into the code to see if there was some kind of path, filename or alike problem: all seemed fine to me. 
As last resort, I decided to try with an explicit `fopen` on a `/tmp/` file, and it failed.
Now, assuming that *on a Unix system nothing can fail writing to `/tmp`*, I decided to dig a little more fouding out that **the problem was caused by `systemd` securying `apache`**.

I found that searching for the files on the `/tmp` space:

```sh
% sudo find /tmp -name 'MY-APPLICATION*'
/tmp/systemd-private-652727091e81431fbb51771e978adf20-apache2.service-V5CzcC/tmp/MY-APPLICATION-messages.log
/tmp/systemd-private-652727091e81431fbb51771e978adf20-apache2.service-V5CzcC/tmp/MY-APPLICATION.log
/tmp/systemd-private-652727091e81431fbb51771e978adf20-apache2.service-V5CzcC/tmp/MY-APPLICATION-OpenSSL.log
/tmp/systemd-private-652727091e81431fbb51771e978adf20-apache2.service-V5CzcC/tmp/MY-APPLICATION-Database.log
```

What I was looking at was `systemd` *redirecting* `apache` to write on a *private temporary folder* instead of the public one.

Since I didn't want that, I needed to find out where this configuration is kept; after understanding [why this has been done](http://0pointer.de/blog/projects/security.html). 
Now, the configuration parameter is called `PrivateTmp` and is hidden in the file `/etc/systemd/system/multi-user.target.wants/apache2.service`:

```sh
# cat /etc/systemd/system/multi-user.target.wants/apache2.service 
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
Environment=APACHE_STARTED_BY_SYSTEMD=true
ExecStart=/usr/sbin/apachectl start
ExecStop=/usr/sbin/apachectl stop
ExecReload=/usr/sbin/apachectl graceful
PrivateTmp=true
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

In order to disable this feature:
1. comment out (with a `#`) the `PrivateTmp` parameter;
2. execute the command `systemctl daemon-reload` to inform `systemd` to reload the configuration;
3. restart the `apache` service.

And that's all folks!
