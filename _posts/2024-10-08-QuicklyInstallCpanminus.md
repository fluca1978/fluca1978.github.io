---
layout: post
title:  "Quickly install App::cpanminus"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
A possible way I learnt today to install `cpanm` when it seems really hard to do!

# Quickly install App::cpanminus

I take for granted that `cpan` **is installed** in *every* *nix system, at least I never found a system when this assumption was false.

Until today.

It seems that some RedHat based distributions, most notably Oracle's, do not ship `cpan`.

And the fact is that I don't use `cpan` at all, preferring `cpanm` (i.e., [App::cpanminus](https://github.com/miyagawa/cpanminus){:target="_blank"}) since I find it simpler and quicker to adopt. But when `App:cpanminus` is not installed, I use `cpan` to install it.

Ok, so having that both are missing, how to install stuff from scratch?

Today I found out a new way to install `cpanm` that I was not aware of: [https://cpanmin.us](https://cpanmin.us){:target="_blank"} is a web site that provides a self-contained Perl script that will install automatically `cpanm`. Note the smarter name in the URL!

How to install `cpanm` with this script? Here it is:
- use `curl` to download the Perl text
- pipe it to your system-wide `perl`
- make Perl aware that the result will be `App::cpanminus`.

For instance:

<br/>
<br/>
```shell
% curl https://cpanmin.us/ | perl - App::cpanminus
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  295k  100  295k    0     0   740k      0 --:--:-- --:--:-- --:--:--  742k
!
! Can't write to /usr/local/share/perl5/5.32 and /usr/local/bin: Installing modules to /home/luca/perl5
! To turn off this warning, you have to do one of the following:
!   - run me as a root or with --sudo option (to install to /usr/local/share/perl5/5.32 and /usr/local/bin)
!   - Configure local::lib in your existing shell to set PERL_MM_OPT etc.
!   - Install local::lib by running the following commands
!
!         cpanm --local-lib=~/perl5 local::lib && eval $(perl -I ~/perl5/lib/perl5/ -Mlocal::lib)
!
--> Working on App::cpanminus
Fetching http://www.cpan.org/authors/id/M/MI/MIYAGAWA/App-cpanminus-1.7047.tar.gz ... OK
Configuring App-cpanminus-1.7047 ... OK
Building and testing App-cpanminus-1.7047 ... OK
Successfully installed App-cpanminus-1.7047 (upgraded from 1.7044)
1 distribution installed

```
<br/>
<br/>


When executing as a non-privileged user, the `cpanm` executable will be installed into the `$HOME` folder:

<br/>
<br/>
```shell
% file ~/perl5/bin/cpanm
/home/luca/perl5/bin/cpanm: Perl script text executable


% ~/perl5/bin/cpanm --version
cpanm (App::cpanminus) version 1.7047 (/home/luca/perl5/bin/cpanm)
perl version 5.032001 (/usr/bin/perl)
...

```
<br/>
<br/>


This is a quick trick to get `cpanm` up and running without having to deal with ordinary `cpan`.
