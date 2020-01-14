---
layout: post
title:  "Raku on FreeBSD"
author: Luca Ferrari
tags:
- raku
- freebsd


permalink: /:year/:month/:day/:title.html
---
How hard can it be to get Raku (Perl 6) on FreeBSD?

# Raku On FreeBSD

I wanted to install [Raku](https://raku.org){:target=_blank} on my FreeBSD server, even if it is not listed as a supported platform. After all, how hard can it be?
<br/>
Unluckily, there's no package or ports about Raku:

```shell
% pkg search raku
p5-WebService-Rakuten-0.05_1   Rakuten WebService API

% pkg search perl6
p5-Bundle-Perl6-0.12_1         Bundle to install Perl6-related modules
p5-Perl6-Builtins-0.0.3_2      Provide Perl 5 versions of the new Perl 6 builtins
p5-Perl6-Export-0.07_2         Implements the Perl 6 'is export(...)' trait
p5-Perl6-Export-Attrs-0.000006 Perl 6 'is export(...)' trait as a Perl 5 attribute
p5-Perl6-Form-0.04_2           Implements the Perl 6 'form' built-in
p5-Perl6-Junction-1.60000_1    Perl6 style Junction operators in Perl5
p5-Perl6-Rules-0.03_1          Implements (most of) the Perl 6 regex syntax
p5-Perl6-Say-0.16_1            Perl 6 say (print, but no newline needed) function
p5-Perl6-Slurp-0.051005        Implements the Perl6 'slurp' built-in
p5-Perl6-Subs-0.05_2           Perl6::Subs - Define your subroutines in the Perl 6 style
```

As you can see, there's no one package that actually ships Raku nor Rakudo-Star.
<br/>
Therefore, the easiest way to install it is to follow the instruction on the official web site and compile it:

```shell
% wget https://rakudo.perl6.org/downloads/star/rakudo-star-2019.03.tar.gz
% tar xfz rakudo-star-2019.03.tar.gz
% cd rakudo-star-2019.03
% sudo perl Configure.pl --gen-moar --gen-nqp --make-install --prefix /opt/raku/2019.03
```

However, the very last step produced a problem and the compilation did stop:

```shell
% sudo  perl Configure.pl --gen-moar  --gen-nqp --make-install --prefix /opt/rakudo/2019.03`
...
The following step can take a long time, please be patient.
/opt/rakudo/2019.03/bin/moar --libpath="blib" --libpath="/opt/rakudo/2019.03/share/nqp/lib" --libpath="/opt/rakudo/2019.03/share/nqp/lib" perl6.moarvm --nqp-lib=blib --setting=NULL --ll-exception --optimize=3 --target=mbc --stagestats --output=CORE.setting.moarvm gen/moar/CORE.setting
Stage start      :   0.000
Stage parse      : 148.369
Stage syntaxcheck:   0.000
Stage ast        :   0.000
Stage optimize   : *** Signal 9

Stop.
make[1]: stopped in /usr/home/luca/rakudo-star-2019.03/rakudo
*** Error code 1

Stop.
make: stopped in /usr/home/luca/rakudo-star-2019.03
Command failed (status 256): make
```

As reported in the [issue I opened](https://github.com/rakudo/star/issues/150){:target=_blank}, the system was running FreeBSD 12.1-p1, with Open JDK 13 and Perl 5.30. 
<br/>
*What the hell was the problem?*
<br/>
I tried to run `gmake` instead of `make`, thinking there could have been some problem with the building system, but it was not (i.e., Raku was smart enough to catch `gmake`). However, that revealed the real problem:

```shell
% sudo gmake
...
The following step can take a long time, please be patient.
/opt/rakudo/2019.03/bin/moar --libpath="blib" --libpath="/opt/rakudo/2019.03/share/nqp/lib" --libpath="/opt/rakudo/2019.03/share/nqp/lib" perl6.moarvm --nqp-lib=blib --setting=NULL --ll-exception --optimize=3 --target=mbc --stagestats --output=CORE.setting.moarvm gen/moar/CORE.setting
Stage start      :   0.000
Stage parse      : 140.010
Stage syntaxcheck:   0.000
Stage ast        :   0.000
Stage optimize   : gmake[1]: *** [Makefile:513: CORE.setting.moarvm] Killed
gmake[1]: Leaving directory '/usr/home/luca/rakudo-star-2019.03/rakudo'
gmake: *** [Makefile:44: rakudo/perl6-m] Error 2
```

The line with `Killed` made me think about system resources, and effectively since I was running that on a pretty small system (only 750 MB or RAM!), I guessed a little more swap space could have helped the process.

## Incrementing the Swap Space on FreeBSD

This is very well documented on the [handbook](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/adding-swap-space.html){:target=_blank}: the idea is to add some disk space to be used as a swap device.
<br/>
First of all, for the record, I have to admit that my system was running with 1GB of swap:

```shell
% swapinfo -m
Device          1M-blocks     Used    Avail Capacity
/dev/ada0p3.eli      1023        0     1023     0%
```

Adding another couple of gigabytes to swap was feasible, therefore I created a 2GB file on a partition where I had some rough space:

```shell
% sudo dd if=/dev/zero of=/postgres/swap0 bs=1m count=2048
```

and therefore the file `/postgres/swap0` was there to hold `2GB` of swap. In the case you are asking yourself why the file was placed under `/postgres` the answer is simpler than what you think: begin this particular system a database host, I have a lot of space on the disk/partition that holds PostgreSQL related data that is, in turn, not surprisingly mounted on `/postgres`. Of course, you can place your swap file wherever you like.
<br/>
Having the file in place, it was time to add an entry on `/etc/fstab`:

```shell
% cat /etc/fstab
...
md99 none swap sw,file=/postgres/swap0,late  0 0
```

As you can see, there's nothing really exciting so far, and I'm following the same suggestions in the [handbook](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/adding-swap-space.html){:target=_blank}. 
<br/>
Last, it is time to activate the swap space:

```shell
% sudo swapon -aq
```
and double check the new swap space is there:

```shell
% swapinfo -m
Device          1M-blocks     Used    Avail Capacity
/dev/ada0p3.eli      1023        0     1023     0%
/dev/md99            2048        0     2048     0%
Total                3071        0     3071     0%
```

OK, everything seems fine now, so let's restart the compilation.

## Compiling Raku

With the new swap space in place, installing Raku was possible:

```shell
% sudo perl Configure.pl --gen-moar --gen-nqp --make-install --prefix /opt/raku/2019.03
...
1 bin/ script [prove6] installed to:
/opt/rakudo/2019.03/share/perl6/site/bin

Rakudo Star has been built and installed successfully.
Please make sure that the following directories are in PATH:
  /opt/rakudo/2019.03/bin
  /opt/rakudo/2019.03/share/perl6/site/bin


Rakudo has been built and installed.
``**

**Great, now my FreeBSD system has Perl 6.d!**
<br/>
Last thing to do is place the directories in my shell PATH, that I do with something like the following:

```shell
% cat .zshrc
...
RAKU_HOME=/opt/rakudo/2019.03/
export PATH=$PATH:${RAKU_HOME}/bin:${RAKU_HOME}/share/perl6/site/bin
```

and last I got Perl 6 running:

```shell
% perl6
To exit type 'exit' or '^D'
> say $*PERL
Perl 6 (6.d)
```


# Conclusions

Installing Raku (Perl 6) on FreeBSD is straightforward, but you may be warned that you need enough RAM (or swap space) to compile it.
<br/>
The [official build instructions](https://rakudo.org/files/star/source){:target=_blank} clearly report that you need at least `1,5 GB` of free memory, so I thougth that with `700 MB` of RAM and `1 GB** of swap I was able to compile, but as shown above, it is better to have more memory available!

<br/>
<br/>
<center>
<img src="/images/posts/raku/raku_swap_space.png" />
</center>
<br/>
<br/>
<br/>

**Have `-Ofun`!**
