---
layout: post
title:  "Using Ansible to install the Oracle Client"
author: Luca Ferrari
tags:
- oracle
- ansible
permalink: /:year/:month/:day/:title.html
---
A simple recipe to install the Oracle client on a remote machine using Ansible.

# Using Ansible to install the Oracle Client

Ansible is a great tool for administering machines, and I love it since it allows me to define *recipes* that will ensure the machines will always be (and behave) the same.

One need I seldomly have, is to install the Oracle client on a remote machine.

There are a few roles out there to the rescue, but for my own needs, I can do it with simple tasks, and in this post I describe how I do it.

## Define what it means to "install Oracle client"

Installing the Oracle client on a Linux machine is actually the task of decompressing a bunch of archives that need to be downloaded from the Oracle official website.

Therefore, what I need to do is to provide a place where to extract the files, extract them and (optionally) set the `ORACLE_HOME` variable so that I can interactively use the installed artifacts.


## Setting up Ansible

First of all, you need the Oracle stuff (archives) on your local (orchestrator) machine.
I place the files into the `files/oracle` folder, so that Ansible will automatically go searching into `files`.

<br/>
<br/>
```shell
% ls -1 files/oracle
instantclient-basic-linux.x64-21.12.0.0.0dbru.zip
instantclient-sdk-linux.x64-21.12.0.0.0dbru.zip
instantclient-sqlplus-linux.x64-21.12.0.0.0dbru.zip

```
<br/>
<br/>

Then I define a bunch of variables in my playbook:

<br/>
<br/>
```yaml
---
- hosts: myoraclehost
  user: luca
  become: yes
  become_method: sudo
  vars:
    oracle_base: /opt/oracle
    oracle_client_major: 21
    oracle_client_minor: 12
    oracle_home: "{{ oracle_base }}/instantclient_{{ oracle_client_major }}_{{ oracle_client_minor }}"
    user_login: luca
```
<br/>
<br/>

The variables should be quite straighforward to understand, but in the following I describe them:
- `oracle_base` is where the archvies will be extracted. Note that I haven't named such directory like *oracle_home* because that is not the final `ORACLE_HOME`;
- `oracle_client_major` and `oracle_client_minor` are the version numbers for the Oracle client I'm going to install;
- `oracle_home` is the effective `ORACLE_HOME` that will be created on the remote machine. Note that the variable is set to a concatenation of the previous variables;
- `user_login` is the name of a user that is going to be configured to have `ORACLE_HOME` into its enviroment, and could be different than the `user` used to play the playbook remotely.


## The tasks

Having defined the previous variables, it is now possible to define the tasks.

The first task is to create the `oracle_base` directory:

<br/>
<br/>
```yaml
    - name: Create Oracle directory
      file:
        path: "{{ oracle_base }}"
        state: directory

```
<br/>
<br/>

Then, it is time to extract the local Oracle stuff into the remote machine. Note that I use the `oracle/{{ item }}` relative path that Ansible will resolve as `files/oracle/{{ item }}`. Note also that I use the Oracle major and minor number variables to find exactly the version of the archives I need.

<br/>
<br/>
```yaml
    - name: Place Oracle libraries in {{ oracle_base }} (ORACLE_HOME = {{ oracle_home }})
      unarchive:
        src: "oracle/{{ item }}"
        dest: "{{ oracle_base }}"
      with_items:
        - instantclient-basic-linux.x64-{{ oracle_client_major }}.{{ oracle_client_minor }}.0.0.0dbru.zip
        - instantclient-sdk-linux.x64-{{ oracle_client_major }}.{{ oracle_client_minor }}.0.0.0dbru.zip
        - instantclient-sqlplus-linux.x64-{{ oracle_client_major }}.{{ oracle_client_minor }}.0.0.0dbru.zip

```
<br/>
<br/>

In the above, a fileglob could have been a better idea to list all the items.


Last, and optionally, I can set up the user profile environment:

<br/>
<br/>
```yaml
    - name: Ensure ORACLE_HOME is loaded into the user's profile
      lineinfile:
        dest: "/home/{{ user_login }}/.profile"
        state: present
        regexp: '^ORACLE_HOME'
        line: 'ORACLE_HOME={{ oracle_home }}'

    - name: Ensure ORACLE_HOME is into the user's PATH
      lineinfile:
        dest: "/home/{{ user_login }}/.profile"
        state: present
        regexp: '^export PATH=$ORACLE_HOME:$PATH'
        line: 'export PATH=$ORACLE_HOME:$PATH'


```
<br/>
<br/>


Clearly, if the user needs another shell profile file, you need to adjust the `dest` accordingly.

# Conclusions

Ansible is a robust way to configure automatically machines, even if it can be a little verbose and noisy.
