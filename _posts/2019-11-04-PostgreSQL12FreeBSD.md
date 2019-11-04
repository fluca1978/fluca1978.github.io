---
layout: post
title:  "PostgreSQL 12 package on FreeBSD"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
PostgreSQL 12 is available as binary package on FreeBSD, but not in the quarterly update.

# PostgreSQL 12 package on FreeBSD

In the case you need to install PostgreSQL 12 on FreeBSD please consider it has not reached the `quarterly` `pkg(1)` update, therefore if you install it via `pkg(1)` you will get *PostgreSQL 12 rc1*. However, in the ports tree, PostgreSQL is clearly at version 12 (release).
<br/>
This behavior is due to the fact that since FreeBSD 12, the default repository for packages is `quarterly`, that in short means packages are older than the ports tree.
<br/>
<br/>
In order to install the official release, a new URL for the FreeBSD repository must be set up. The repository URL is placed into the file `/etc/pkg/FreeBSD.conf`:

```shell
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/quarterly",
  mirror_type: "srv",
  signature_type: "fingerprints",
  fingerprints: "/usr/share/keys/pkg",
  enabled: yes
}
```

The `pkg(1)` configuration allows the overriding of the default URL placing a file `/usr/local/etc/pkg/FreeBSD.conf` that overrides the properties of the above, so with the content:

```sql
FreeBSD: {
  url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest"
}
```

After that, the repository can be updated and new packages will be available. Therefore, run: 

```shell
% sudo pkg update
% sudo pkg install postgresql12-client-12 \
                   postgresql12-contrib-12 \
                   postgresql12-docs-12 \
                   postgresql12-plperl-12 \ 
                   postgresql12-server-12
```

