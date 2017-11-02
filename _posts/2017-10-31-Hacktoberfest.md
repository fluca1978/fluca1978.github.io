---
layout: post
title:  "Hacktoberfest 2017"
author: Luca Ferrari
tags:
- Hacktoberfest
- Open Source
- Perl
- Sqitch
permalink: /:year/:month/:day/:title.html
---
Another small contribution to the Open Source world!

## Hacktoberfest 2017
-----

This is the second year I take place in the **[Hacktoberfest](https://hacktoberfest.digitalocean.com/)**, and I'm really proud to what I've done without any regard to the fact that my Pull Requests have been accepted or not.

So what is *Hacktoberfest*?
Well, it is a contest that takes place during the month of October and involves any developer around the world that have time, and most notably
the **will to contribute to Open Source**. The idea is simple: place at least four Pull Request on [github](https://github.com)
before October ends and you're done!

[Last year](https://fluca1978.github.io/hacktoberfest/) I was a tiny and scared developer, well I've already performed a few CPAN-PR
but I was not able to contribute really aggressively to other people's projects. Anyway, [I did 9 PRs and got the shrew](https://fluca1978.github.io/hacktoberfest-shrew/).

This year I decided to work more intensively to help projects I care and I like.
Two in particular hit my wish list: **perlbrew** and **rakudobrew**. Let's be honest here: I use the former much more than the latter,
and I don't believe I'm a so proficient Perl developer, anyway, I took time to do some possibly useful coding.

So here's the list of my PRs:
1. [*perlbrew clone-modules*](https://github.com/gugod/App-perlbrew/pull/564): a command to automatically clone modules
from one Perl installation to another one, something I do regularly when I install a new(er) version of Perl.
2. [*perlbrew sort list and available versions*](https://github.com/gugod/App-perlbrew/pull/565): this is something I worked before, but I guess
it was out of the past Hacktoberfest. However, I implemented a sorting routine to display newer version on the top of the list of both
*available* and *list* commands consistently. This should solve some ticket complains.
3. [*perlbrew show installation date*](https://github.com/gugod/App-perlbrew/pull/566): this is a kind of *troophy patch*, since it is not
so widely useful. The idea is to show the ```ctime``` of the ```perl``` executable in order to let the user know how aged is
her installation.
4. [*perlbrew installer*](https://github.com/gugod/App-perlbrew/pull/568): minor changes to automated installer script in order
to use variables and loops consistently.
5. [*perlbrew shell profile instructions*](https://github.com/gugod/App-perlbrew/pull/569): solves some ticket complains. The idea
is to understand which shell the user is running and provide more accurate information about which profile file must to be edited
in order to make ```perlbrew``` working.
6. [*perlbrew --force and --yes*](https://github.com/gugod/App-perlbrew/pull/570): solves some ticket complains making the two options
a synonim each other and simplifying the command line usage.
7. [*perlbrew download failure*](https://github.com/gugod/App-perlbrew/pull/571): since different external programs can be used to
download a new perl tarball, and since each program has different exit codes, I decided to place a callback on each program hash entry
in order to simplify the code for handling error situations.
8. [*rakudobrew shell configuration*](https://github.com/tadzik/rakudobrew/pull/123): solves some ticket complaining trying to
provide better information to the user configuration depending on the shell she is using.
9. [*rakudobrew nuke help*](https://github.com/tadzik/rakudobrew/pull/124): solves some ticket complaining providing help
for the ```nuke``` command and, more in general, for every command that requires a specific version to be passed as command line argument.
10. [*rakudobrew general command help*](https://github.com/tadzik/rakudobrew/pull/125): solves a ticket and also provides a kind of *platform*
for specifying command help. The idea is to use ```Pod::Usage``` to *read* the POD documentation at the very end of the script itself,
so that it can provide *sub command help* in a more accurate way.
11. [*sqitch italian translation*](https://github.com/theory/sqitch/pull/357): this is, well, quite trivial, even if required
a lot of time to translate the whole set of messages of this great database management application. This is, so far, the second or third
translation I try and the first one that I complete!
12. [*rakudobrew minor changes*](https://github.com/tadzik/rakudobrew/pull/126): this started as a simple adjusting of the ```run```
subroutine, but then submitting a PR with a single commit seemed a little stupid to me, so I tried to apply a few small rules of
*best practices* such as avoiding ```return undef```, checking that a list return was applied in list context and ensuring
un-interpolable strings whenever possible.
13. [*Perl 6 documentation*](https://github.com/perl6/doc/pull/1594): a short and sweet introduction to *precedence dropping*, something I foundmyself difficult to learn and therefore I thought it was useful for other developers to find already cited.
14. [*shame on me for Perl6 documentation!*](https://github.com/perl6/doc/pull/1601): after the previous PR has been merged I noted that the
web page for the *syntax* documentation was rendering wrong: I mispelled a ```head2``` POD title to ```head 2``` and nobody noticed! So this
is not a real PR that counts, rather a quick and dirty fix for a horrible mistake I did introduced!
15. [*Perlbrew reverse sorting customization*](https://github.com/gugod/App-perlbrew/pull/575): due to an issue, my previous patch about
sorting output could not meet all users' wills, so I added a ```--reverse``` option to allow users to decide how they
want the sorting of Perl versions.

Here you can [check my Hacktoberfest status](https://hacktoberfestchecker.herokuapp.com/?username=fluca1978).

As a side note I have to say that, being a [magit](https://github.com/magit/magit) user, I took advantages of this great program
to overtake some git problems and, on the other hand, this experience made me a better magit user as well as a github user.
It is interesting also to note that, differently from the past year, I did not *rebase* my branches. It is not that my commits are
perfect, it is just that my [Fossil](https://www.fossil-scm.org/index.html/doc/trunk/www/index.wiki) experience teached me to value
every single  commit and remember my mistakes. And I believe there's nothing wrong in showing to the world my mistakes, what it does matter, after all, is the *full diff* that can lead to the contribution.

One thing I have to admit this Hacktoberfest did to me is that **it took me back to my desk, at eveningn, developing some new feature**. That was
a thing I did not do anymore, I try to sped the less time as possible in front of my computer when at home, but this time I did enjoy being back
at evening, with my cat near me (unluckily it was not enough cold to start up my fireplace, that would make it a perfect hacking session).


But wait a minute: how did all the above go?
Well, the first PRs to be evaluated have been, of course, the Perl 6 documentation ones and that reminded me how important and organized
the documentation is. Well, I have to admit I was happy to see [how strict it was the process to get the PR accepted](https://github.com/perl6/doc/pull/1594/commits/69d7951fc52efc455c3885677c95a7f121fed40d) while looking at the documentation
did not make me feel the same. I mean, the documentation seems somehow less structured with regard to the Perl 5 one, but
with such a PR process it will catch up very quickly. Also please note that whithin three hours the repository passed from my [requeste numbered 1594](https://github.com/perl6/doc/pull/1594) to the fixing one [numbered 1601](https://github.com/perl6/doc/pull/1601), that is the project is
really alive!

The [Sqitch translation PR](https://github.com/theory/sqitch/pull/357) provided me the *commit-bit* to the repository, and that is something somehow scarying because (i) I have to keep the translation up to date and (ii) I have the possibility to mess up the official repository with a wrong push!

The Perlbrew project slowly merged the followings: the [clone modules command](https://github.com/gugod/App-perlbrew/pull/564),  [sorting of Perl versions from the newest to the oldest](https://github.com/gugod/App-perlbrew/pull/565), [keeping 'yes' and 'force' coherent](https://github.com/gugod/App-perlbrew/pull/570), [callable function to manage download return code](https://github.com/gugod/App-perlbrew/pull/571).

Therefore, so far, the situation is:

1. [*perlbrew clone-modules*](https://github.com/gugod/App-perlbrew/pull/564) **merged!**
2. [*perlbrew sort list and available versions*](https://github.com/gugod/App-perlbrew/pull/565) **merged!**
3. [*perlbrew show installation date*](https://github.com/gugod/App-perlbrew/pull/566) **merged!**
4. [*perlbrew installer*](https://github.com/gugod/App-perlbrew/pull/568) *merged*
5. [*perlbrew shell profile instructions*](https://github.com/gugod/App-perlbrew/pull/569) **merged!**
6. [*perlbrew force and yes*](https://github.com/gugod/App-perlbrew/pull/570) **merged!**
7. [*perlbrew download failure*](https://github.com/gugod/App-perlbrew/pull/571) **merged!**
8. [*rakudobrew shell configuration*](https://github.com/tadzik/rakudobrew/pull/123) **merged!**
9. [*rakudobrew nuke help*](https://github.com/tadzik/rakudobrew/pull/124) *waiting*
10. [*rakudobrew general command help*](https://github.com/tadzik/rakudobrew/pull/125) *waiting*
11. [*sqitch italian translation*](https://github.com/theory/sqitch/pull/357) **merged and repo invitation!**
12. [*rakudobrew minor changes*](https://github.com/tadzik/rakudobrew/pull/126) *waiting*
13. and 14. [*Perl 6 documentation*](https://github.com/perl6/doc/pull/1594) and [*shame on me for Perl6 documentation!*](https://github.com/perl6/doc/pull/1601) **merged!**
15. [*Perlbrew reverse sorting customization*](https://github.com/gugod/App-perlbrew/pull/575): *waiting*
