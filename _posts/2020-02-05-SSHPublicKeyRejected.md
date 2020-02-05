---
layout: post
title:  "SSH Not Accepting Public Key"
author: Luca Ferrari
tags:
- linux
- ssh
permalink: /:year/:month/:day/:title.html
---
A noob error in the configuration of my home became quickly an annoying password prompt!

# SSH Not Accepting Public Key

I found that one server on which I work day-by-day was refusing my SSH public key login.
<br/>
I thought it was related to the fact that, a couple of days before, I changed my login password, but after all login passwords have nothing to do with public key, but that was a puzzling coincidence.
<br/>
However, after a quick check with other server, I discovered that the latter were working fine, so it was not a problem with my account nor my password nor my public key, *it was a problem with that specific target server*!
<br/>
<br/>
The first step, as usual, was to launch a connection in very verbose mode:

```shell
% ssh luca@miguel -vvv
...
debug1: Offering public key: /home/luca/.ssh/id_rsa RSA SHA256:ZfOdr+nbcEFprgtsJAQ58hggjlpR1LPr9VuPMnESTOA
debug3: send packet: type 50
debug2: we sent a publickey packet, wait for reply
debug3: receive packet: type 51
debug1: Authentications that can continue: publickey,keyboard-interactive
debug1: Trying private key: /home/luca/.ssh/id_dsa
debug3: no such identity: /home/luca/.ssh/id_dsa: No such file or directory
debug1: Offering public key: /home/luca/.ssh/id_ecdsa ECDSA SHA256:frgH+DO2U3AhPLw7KWBWUXnFQ/CmfghG/7tFylgW/8g
debug3: send packet: type 50
debug2: we sent a publickey packet, wait for reply
debug3: receive packet: type 51
debug1: Authentications that can continue: publickey,keyboard-interactive
...
```

So, the keys were effectively offered, but the server was not considering them as valid.
<br/>
So I logged into my server and double checked the `.ssh` directory, that apparently had right permissions and files in there:

```shell
% ls -l .ssh 
total 20
-rw-------  1 luca  luca   962 Oct 28 08:56 authorized_keys
-rw-r--r--  1 luca  luca   393 Mar  1  2019 id_rsa.pub
``**

<br/>
So what's next?
<bR/>
**Home directory**! I checked my home directory and find out that someone (the server administrator?) has changed the permissions to everyuser!
<br/>

```shell
% ls -ld $HOME
drwxrwxrwx  32 luca  luca  4096 Feb  5 11:29 /home/luca
```

Ah ah!
<br/>
Let's turn my home directory to the right set of permissions and then SSH will work with public key again.

```shell
% chmod 700 .
```

<br/>
Now I can go back working, but before that, let's just blame the admins for this waste of time!
