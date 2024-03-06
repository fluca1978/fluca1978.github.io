---
layout: post
title:  "The curious case of the 'undefined subroutine'"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
A strange behavior that appeared on an application of mines.

# The curious case of the 'undefined subroutine'

I was periodically running an application that has to mangle some huge dataset, so I wrapped into a `cron` (yes, one day I will learn about `systemd` timers!), and let it run, monitoring sometime the progress.

After several hours, the progress of the application stopped, even if everything semmed to run fine.
The application begun to report an uncaught exception: `Undefined subroutine &main::duration called at ...`. The `duration` function was an imported function from `Time::Duration`, so what was going wrong?

TL;DR: **it was not a `Time::Duration` fault!** The module is working just fine, don't worry if you are using it too.
However, in the beginning, I thought I somehow changed something within the run-time environment: could an update of the module changes the exported set of subroutines? It was the first thing I checked, but nothing was updated, nor the subroutine was requiring an implicit export.

So, what was going wrong here?

I quickly checked my `git` working tree, thinking I somehow switched the branch to a non-production ready one, but again, this was not the cause.

So I created a one liner to test the module:

<br/>
<br/>
```shell
% perl -MTime::Duration -E 'say duration( 100 );'
Undefined subroutine &main::duration called at -e line 1.
```
<br/>
<br/>


What the hell was happening here? I asked for some help on IRC, that clearly turned my mind on.
A user asked the perlbot to run the same one liner above, and the bot claimed it didn't have a `Time::Duration` module on `@INC`.

Let's repeat: `@INC` was missing `Time::Duration`. So I removed my `Time::Duration` module (via `cpanm --uninstall Time::Duration`) and launched again the one line, **obtaining the very same result: the routine was undefined even if the module was missing on my system!**
This was an amazing clue to find the solution: somewhere there was a `Time::Duration` module still surviving.

Therefore, I printed out `@INC` with the classic `perl -E 'say join( "\n", @INC);'` and used `find` to find out a `Duration.pm` within each directory.
And I discovered that within the `lib` directory of the application itself, there was a `Time/Duration.pm` module.

What the hell?

Well, removing such **empty** module from the filesystem fixed the problem!
But why was the issue appeared in first place? Well, it turned out that I misrun a `Dist::Zilla` command: `dzil add Time::Duration`. This created the empty module into my application, that in turn picked up the module *in front* of the official one, and being the former empty, the subroutine was clearly missing from there.

Shame on me!


## IRC Chat Logs

If you mind to read all the discussion about this, here it is:

<br/>
<br/>
```
<fluca1978> for some reason, my application that uses Time::Duration stopped
	    working, and I found that now: perl -MTime::Duration -E 'say
	    duration( 100 );'  [08:17]
<fluca1978> Undefined subroutine &main::duration called at -e line 1.
<fluca1978> perl is 5.38, and Time::Duration is (obviously) installed, what am
	    I missing here?

<vague> Try -MTime::Duration=duration  [08:22]
<vague> I would suspect the method isn't exportedc by default

<fluca1978> vague: already done  [08:23]
<fluca1978> perl -MTime::Duration=duration -E 'say later( 100 );'
<fluca1978> Undefined subroutine &main::later called at -e line 1.
<fluca1978> the methods are all exported by default, and I've tried also to
	    reinstall (cpanm reinstall Time::Duration), but I cannot figure
	    out what happened. It worked as far as one day ago

<vague> fluca1978, you imported duration but used later  [08:32]

<fluca1978> vague: yeah, sorry, wrong copy and paste
<fluca1978> vague: perl -MTime::Duration=duration -E 'say duration( 100 );'
<fluca1978> Undefined subroutine &main::duration called at -e line 1.
<vague> eval5.36: use Time::Duration; [duration(100)]  [08:35]
<perlbot> vague: ERROR: Can't locate Time/Duration.pm in @INC (you may need to
	  install the Time::Duration module) (@INC contains:
	  $PERLS/5.36.3/lib/site_perl/5.36.3/x86_64-linux
	  $PERLS/5.36.3/lib/site_perl/5.36.3
	  $PERLS/5.36.3/lib/5.36.3/x86_64-linux $PERLS/5.36.3/lib/5.36.3) at
	  (IRC) line 1. BEGIN failed--compilation aborted at (IRC) line 1.
<vague> The pod says all functions are exported by default so maybe something
	changed in 5.38  [08:37]

<fluca1978> vague: after uninstall (cpanm --uninstall Time::Duration), the
	    following prints "ok", so it seems the module is loaded somewhere
	    else: perl -E 'use Time::Duration; say q/ok/;'  [08:46]
<fluca1978> vague: after inspecting @INC, I found out there was a
	    Time/Duration.pm wihin a project lib directory, that was the
	    problem, thanks!  [08:48]
<vague> Ah, good you  figured it out :)  [08:51]

```
<br/>
<br/>
