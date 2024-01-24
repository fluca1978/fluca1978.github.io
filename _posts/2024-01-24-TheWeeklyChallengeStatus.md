---
layout: post
title:  "How much did I do on The (Perl) Weekly Channelge?"
author: Luca Ferrari
tags:
- perlweeklychallenge
- perl
- raku
permalink: /:year/:month/:day/:title.html
---
Where am I on the Perl Weekly Challenge?

# How much did I do on The (Perl) Weekly Channelge?

I started the **[Perl Weekly Challenge](https://theweeklychallenge.org/challenges/){:target="_blank"}** (nowdays simply called *The Weekly Challenge)* back in the beginning of 2020.

I started to do the challenge because I wanted to learn more **Raku (aka Perl 6)** and since, *unluckily*, I did not have any real chance to use it on my day-to-day job (and, sadly, I don't have even today), that seemed to me a good way to stay up to date and learn the language.

My initial commits were in Raku only.

After a while, I decided to introduce also **PL/Perl**, the Perl-ish way of doing things in PostgreSQL. Therefore, I was staying up to date with my general Perl knowledge and also gaining more and more experience in solving problems within PostgreSQL. Clearly, I also added **PL/PgSQL**, the *native* procedural language of PostgreSQL to remain up to date with such language too.

For a couple of years, I went with only those languages, and it was nice and an excellent way to learn more and more about everyone of the above.

In recent days, a few months ago, I introduced also **Python** because I want to better learn this language. Last, a few weeks ago, I also introduced **PL/Java** because I don't develop in Java anymore on a daily basis, and I want to keep my knowledge in good shape. But Java itself could be boring, so I decided to challenge me with the more difficult *Java in PostgreSQL* language (the difficulty is about the build and type-mapping).


So, where am I today in the Perl Weekly Challenge?

## My PWC Numbers

With my surprise, I'm within the **third place** of the *Team Leaders* at the time of writing.

<center>
<br/>
<img src="images/post/perlweeklychallenge/pwc_team_leaders_2024_01.png" alt="Team Leaders as of January 2024" />
<br/>

<br/>
<img src="images/post/perlweeklychallenge/pwc_team_leaders_2024_01b.png" />
<br/>
</center>


So far I've pushed **2192** pieces of code and blog posts, over a total counting of **178** unique weekly challenges.
With the exclusions of a few periods, mainly due to my eyes surgeries, I've done all the weekly challenges in a row.

It is possible to find out all my solutions in one place [within one of my GitHub repositories](https://github.com/fluca1978/fluca1978-coding-bits/tree/master/PWC){:target="_blank"}.

Counting the lines of code, for example via `[App::Cloc](https://metacpan.org/dist/App-cloc/view/bin/cloc){:target="_blank"}` results in the following report:

<br/>
<br/>
```shell
% cloc ~/git/fluca1978-coding-bits/PWC
     752 text files.
     750 unique files.
       3 files ignored.

github.com/AlDanial/cloc v 1.76  T=1.06 s (708.8 files/s, 23104.4 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Perl                           515           3439           2037           9918
SQL                            198           1238            972           5603
Python                          32            319            259            541
Java                             3             21             21             41
Bourne Shell                     1              2              0              5
-------------------------------------------------------------------------------
SUM:                           749           5019           3289          16108
-------------------------------------------------------------------------------

```
<br/>
<br/>

therefore assuming **16 thousand lines of code** and a grand total of more than 21 thousand of lines. The blog posts need to be added to these numbers to get the final result as counted by the Weekly Challenge.

> Note that App::cloc does not recognize the .raku extension, as well as .plperl, so in order to make it happy, I temporary renamed files in the appropiate way.


## How do I prepare for a Weekly Challenge?

After a few challenges, I created a shell script, that I named `pwc.sh` that creates all the skeleton structure for me:
- it fetches the updates from the original repository;
- it creates the basic scripts for every language within my directory;
- it prepares the blog references for the challenge.

The script also allows me for lazily merge the solutions into my personal GitHub repository, and I do it seldomly.

Therefore, when I want to start a new challenge, I can simply do:

<br/>
<br/>
```shell
% pwc.sh create 253
```
<br/>
<br/>

and start coding. When I want to merge all the solutions [within one of my GitHub repositories](https://github.com/fluca1978/fluca1978-coding-bits/tree/master/PWC){:target="_blank"}, I can simply do:

<br/>
<br/>
```shell
% pwc.sh sync
```
<br/>
<br/>

On the blogging side, I use a template within Emacs `YASnippet`, alrteady set up for me to just be filled with the code and its explaination. If you follow my solutions, you will see that all the posts are pretty much the same, with the same title, order, structures, and so on.


It is interesting the way I create the skeletons of every source code solution: in the beginning I was using a shell *here document* approach, but nowdays I use `Template::Toolkit` and `ttree` to generate the stubs. I'm thinking to move over to `tpage` to have a better control and less repetitions among skeletons for the first and second task. Thanks to this, I can start coding in a very short time and with a very consistent environment.

## In which order do I write my challenges?

I always start from Raku, solving both the challenges. Then I move forward to PL/Perl, because implementing what I've done in Raku is often simple, being both Perl-ish languages.

Then I do PL/PgSQL, which has often if not always a very verbose approach.

Then it is the turn of Python, and last comes PL/Java.

There is nothing special about the order I use the above languages, it is a matter of habits.

## Why am I calling it *Perl Weekly Challenge*?

During the time, the *Perl Weekly Challenge* changed its name into **The Weekly Challenge**, due to the fact that people (not only me, of course!) were submitting solutions in foreign languages, that means not in Perl. However, since I started it as *Perl Weekly Challenge* and my main focus is on Perl and Raku, I still keep it named as such in my git references and directory structure.


## What gives me back doing all these challenges?

In the beginning, I was doing the challenges to learn Raku.

Then to train in other languages I don't use on a daily basis at work. Then things changed, and now I use such languages and I don't use others, so I keep my shape doing as much languages as possible in a trade-off between time and effort.

But for sure, an interesting side effect of the above is that I have a better and cleaer comparison of the languages I use. For example, I can recognize the beauty of Perl and Raku against the ugliness of Java and Python, at least according to me. And I can quickly get a measure about the effort to "migrate" even a simple program from one language to the other.


# Conclusions

The weekly challenge is something that really improved my developer abilities and knowledge. It could seem, at glance, that the tasks assigned every week are unlikely to be found in a *real workload*, but the fact is that solving simple puzzles always helps you on focusing on small and self explainatory solutions, something I appreciate also in my *real life as developer*.

I strongly encourage everyone to invest some time in doing the weekly challenges, and I advocate this project in many online palces where people is asking on ways to improve their skills in a particular language.
