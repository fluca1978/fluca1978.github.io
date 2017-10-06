---
layout: post
title:  "Hacktoberfest 2017"
author: Luca Ferrari
tags:
- Hacktoberfest
- Open Source
permalink: /:year/:month/:day/:title.html
---
Another small contribution to the Open Source world!

## Hacktoberfest 2017
-----

This is the second year I take place in the **[Hacktoberfest](https://hacktoberfest.digitalocean.com/)**, and I'm really proud to what I've donewithout any regard to the fact that my Pull Requests have been accepted or not.

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


Here you can [check my Hacktoberfest status](https://hacktoberfestchecker.herokuapp.com/?username=fluca1978).

As a side note I have to say that, being a [magit](https://github.com/magit/magit) user, I took advantages of this great program
to overtake some git problems and, on the other hand, this experience made me a better magit user as well as a github user.
It is interesting also to note that, differently from the past year, I did not *rebase* my branches. It is not that my commits are
perfect, it is just that my [Fossil](https://www.fossil-scm.org/index.html/doc/trunk/www/index.wiki) experience teached me to value
every single  commit and remember my mistakes. And I believe there's nothing wrong in showing to the world my mistakes, what it does matter, after all, is the *full diff* that can lead to the contribution.
