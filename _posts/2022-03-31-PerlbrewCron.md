---
layout: post
title:  "Running Perl thru crontab and perlbrew"
author: Luca Ferrari
tags:
- perl
- linux
permalink: /:year/:month/:day/:title.html
---
How to configure `crontab(5)` to properly run Perl via `perlbrew`.

# Running Perl thru crontab and perlbrew

[Perlbrew](https://perlbrew.pl){:target="_blank"} is **amazing**! It allows you to install different Perl versions within your home directory, tune such installation (via `cpanm`) and **be free to use modern Perl and related libraries** even on those systems you don't have control over and that runs, as an example, Perl `5.16`-something!
<br/>
For me, [Perlbrew](https://perlbrew.pl){:target="_blank"} has been a real changer because now I can develop and run a lot of automated tasks written in one of my favourite language without any worry or need to ask a system administrator to install, upgrade and tune something for me.
<br/>
<br/>
But then I want to push it further, and *schedule* Perl task execution via `crontab(5)`.
<br/>
That should sound easy, but it is not as it may seem. The fact is that Perlbrew requires some trick to instrument your shell to run the version of Perl you want to, and `crontab(5)` does not always respect your login shell artifacts and tuning.

## `perlbrew exec` as a way to quickly setup a `crontab(5)` task

Perlbrew includes a particular command, named `exec` that allows you to run a program using the Perlbrew configuration (i.e., the Perl you have `switch`ed to). Therefore, my crontab could look like the following one:


<br/>
<br/>

``` shell
ORACLE_HOME=$HOME/instantclient_18_3
SHELL=/bin/bash
# min hour dom month dow
35     22   *   *     1-6 $HOME/perl5/perlbrew/bin/perlbrew exec perl $HOME/my_script_perl.pl -r -v -a /tmp
```
<br/>
<br/>


as you can see, in the beginning of the crontab I set some variables, most notably the `ORACLE_HOME` because I could need to run scripts that use `DBD::Oracle`, but that's an example.
<br/>
The important thing to note is **the absolute path to `perlbrew`**, then the `exec` command and the full command line to the script I want to execute, including various options.
<br/>


## Who is running what?

When the scheduler will execute, you are going to see **two** (or more) processes that seems the same:

<br/>
<br/>

``` shell
$ ps
...
luca  2292  1.1  0.2 157504 11948 ?        Ss   12:49   0:00 /usr/bin/perl /home/luca/perl5/perlbrew/bin/perlbrew exec perl /home/luca/my_script_perl.pl -r -v -a /tmp
luca  2293  1.8  0.6 395008 25344 ?        S    12:49   0:00 perl /home/luca/my_script_perl.pl -r -v -a /tmp

```
<br/>
<br/>

The first process is the execution of `perlbrew` (note the usage of the *default* `perl`). The second process is the sub-process run by `perlbrew exec`.


# Conclusions

With a little effort, it is possible to avoid wrapping scripts and use `perlbrew` directly into `crontab(5)`.
<br/>
Now, it possible to automate pretty much everything with the power of Perl at our fingertips!
