---
layout: post
title:  "Ansible and FreeBSD: cannot start from scratch?"
author: Luca Ferrari
tags:
- ansible
- freebsd
permalink: /:year/:month/:day/:title.html
---
My baby-steps in the Ansible world.

# Ansible and FreeBSD: cannot start from scratch?


The first failure I encountered was:

<br/>

```shell
fatal: [venkman]: FAILED! => {"changed": false, "module_stderr": "Shared connection to venkman closed.\r\n", "module_stdout": "/bin/sh: /usr/local/bin/python: not found\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 127} 
```

<br/>
The important part is `/usr/local/bin/python: not found`, that tells exactly what the problem is: **FreeBSD installs a minimal system that does not include `python`**.
<br/>
The solution is, as you can imagine, to install the `python` executable, for example by means of:

```shell
# pkg install python sudo
```

<br/>
As you can see, you need to install also `sudo` in order for Ansible to work properly.

*The annoying part is that, in order to setup the machine, you need to setup a part of the machine!*
<br/>
Anyway, once `python` and `sudo` are installed, I tried to execute again the playbook, but another failure raised:


```shell
fatal: [venkman]: FAILED! => {"changed": false, "module_stderr": "Shared connection to venkman closed.\r\n", "module_stdout": "sudo: a password is required\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}
```

<br/>
You need to ensure that your own user is able to *become* (in Ansible terminology) without a password, that is you need to edit `sudoers` appropriately.

# Summary

It seems that, while Ansible is a great tool, it is not possible to use to configure a machine from scratch, and this is really true for FreeBSD, where the system does not install a lot of *default* software that is available on many Linux distributions.
