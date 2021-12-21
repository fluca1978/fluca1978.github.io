---
layout: post
title:  "Using doas in Ansible"
author: Luca Ferrari
tags:
- ansible
- doas
- openbsd
- freebsd

permalink: /:year/:month/:day/:title.html
---
Ansible is able to use different privilege escalation methods, including `doas`.

# Using `doas` in Ansible

I'm moving from `sudo` to `doas` in pretty much all my installations.
The reason is that `doas` is simpler to configure and manage, and I trust the [OpenBSD](https://openbsd.org){:target="_blank"} team with regard to producing clean and functional pieces of code.
<br/>
<br/>
But how to automate tasks thru `ansible` using `doas` instead of the well established `sudo`?
<br/>
Well, Ansible allows the definition of an `ansible_become_method` variable, that can be specified to isntrument `ansible` on how to gain privileges when executing a task.
Here I present a few available options to use `doas` while running `ansible`.

## Specifying `become-method` on the command line

One way to quickly test your playbook and your configuration for using `doas` is to run the playbook with a different command line option:

<br/>
<br/>

``` shell
% ansible-playbook -l ghostbusters --become-method=doas  FreeBSD.yml
```
<br/>
<br/>

In the above, I'm running the `FreeBSD.yml` playbook against the `ghostbustes` group of hosts, and **I'm specifying the `become-method` variable as to use `doas`**.
<br/>
Another option, from the command line, is to override the internal variable `ansible_become_method`, such as:

<br/>
<br/>

``` shell
% ansible-playbook -l venkman  FreeBSD.yml --extra-vars "ansible_become_method=doas"
```
<br/>
<br/>

## Specifying the `ansible_become_method` in the playbook

A simple but not very scalable, according to me, approach, is to specify the particular variable `ansible_become_method` in the playbook. The variable can be specified on a single task basis, or as a general variable, so for example in your playbook you can place it into the `vars` section:


<br/>
<br/>

``` shell
- hosts: freebsd
  vars:
    ansible_become_method: doas
...
```
<br/>
<br/>

This means you don't have to specify anymore any particular flag on the command line.


## Specifying `ansible_become_method` on a per-host basis

A more beautiful approach, according to me, is to specify the `ansible_become_method` on a per-host basis.
In my inventory file `hosts`, I do have something like:

<br/>
<br/>

``` shell
[freebsd]
miguel  ingress_ipv4=192.168.222.123
venkman ingress_ipv4=192.168.222.13  ansible_become_method=doas
```
<br/>
<br/>

So my group `freebsd` has two hosts, where only `venkman` will use `doas` as a pribvilege escalation method.


# Conclusions

Ansible is a very flexible tool, that allows automation even using privilege escalation methods that are *still* new!
