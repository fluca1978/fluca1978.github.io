---
layout: post
title:  "Installing PostgreSQL 16 (development) on Rocky Linux 9: the Perl::IPC::Run problem"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perl
permalink: /:year/:month/:day/:title.html
---
A possible solution to a common problem

# Installing PostgreSQL 16 on Rocky Linux 9: the Perl::IPC::Run problem

Today I was preparing a new machine, based on Rocky Linux 9, for some development activity.
I was installing PostgreSQL 16 and the development stuff I need, so I was executing (after having imported the PGDG repository), the usual:

<br/>
<br/>
```shell
% sudo dnf install postgresql16.x86_64 \
                   postgresql16-contrib.x86_64 \
				   postgresql16-devel.x86_64 \
				   postgresql16-libs.x86_64 \
				   postgresql16-plperl.x86_64 \
				   postgresql16-server.x86_64

...
Error:
 Problem: cannot install the best candidate for the job
  - nothing provides perl(IPC::Run) needed by postgresql16-devel-16.1-2PGDG.rhel9.x86_64 from pgdg16
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)

```
<br/>
<br/>

Apparently I'm not able to find out a `Perl-IPC-Run` module on the Rocky Linux repositories, nor in the `epel_release` ones.

The correct way is to enable the `crb` repository:

<br/>
<br/>
```shell
% sudo dnf config-manager --set-enabled crb
```
<br/>
<br/>

And that's it:

<br/>
<br/>
```shell
% dnf search Perl-IPC-Run

==================================== Name Matched: Perl-IPC-Run ====================================
perl-IPC-Run.noarch : Perl module for interacting with child processes
perl-IPC-Run3.noarch : Run a subprocess in batch mode

```
<br/>
<br/>


Another approach is to install it the *Perl way!*

I prefer to use `cpanm` as Perl package manager nowdays, but `cpan` and others work equally well:


<br/>
<br/>
```shell
% sudo dnf install perl-App-cpanminus.noarch

% sudo cpanm IPC::Run
--> Working on IPC::Run
Fetching http://www.cpan.org/authors/id/T/TO/TODDR/IPC-Run-20231003.0.tar.gz ... OK
Configuring IPC-Run-20231003.0 ... OK
Building and testing IPC-Run-20231003.0 ... OK
Successfully installed IPC-Run-20231003.0 (upgraded from 20200505.0)
1 distribution installed

```
<br/>
<br/>
