---
layout: post
title:  "Installing PostgreSQL on FreeBSD via Ansible"
author: Luca Ferrari
tags:
- postgresql
- freebsd
- ansible
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
My very simple attempt at keeping PostgreSQL up-to-date on FreeBSD machines.

# Installing PostgreSQL on FreeBSD via Ansible

I'm slowly moving to [Ansible](https://www.ansible.com/) to manage my machines, and one problem I'm trying to solve at best is how to keep PostgreSQL up-to-date.
<br/>
In the case of FreeBSD machines, `pkgng` is the module to use, but in the past I was used to this very simple playbook snippet:


```yaml
{% raw %}
- name: PostgreSQL 11
  become: yes
  with_items:
    - server
    - contrib
    - client
    - plperl
  pkgng:
    name: postgresql11-{{ item }}
    state: latest
{% endraw %}    
```

However, there is a very scarign warning message when running the above:

```shell
TASK [PostgreSQL 11] 
[DEPRECATION WARNING]: Invoking "pkgng" only once while using a loop via squash_actions is deprecated. Instead of using a loop to supply multiple 
items and specifying `name: "postgresql11-{{ item }}"`, please use `name: ['server', 'contrib', 'client', 'plperl']` and remove the loop. This 
feature will be removed in version 2.11. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
```

That's easy to fix, but also annoying (at least to me), because I have to change the above snippet to the following one:

```yaml
- name: PostgreSQL 11
  become: yes
  pkgng:
    name:
      - postgresql11-server
      - postgresql11-contrib
      - postgresql11-client
      - postgresql11-plperl
    state: latest
```

So far, the better solution I've found that helps me keep readibility is to use a variable to hold the PostgreSQL version I want and the list of packages I need:

```yaml
{% raw %}
vars:
  pg_version: 11
  pg_components:
    - postgresql{{ pg_version }}-server
    - postgresql{{ pg_version }}-contrib
    - postgresql{{ pg_version }}-client
    - postgresql{{ pg_version }}-plperl


tasks:
    - name: PostgreSQL {{ pg_version }}
      become: yes
      pkgng:
        name: "{{ pg_components }}"
        state: latest
{% endraw %}        
```
