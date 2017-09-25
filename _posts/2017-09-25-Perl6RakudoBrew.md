---
layout: post
title:  "rakudobrew to the rescue!"
author: Luca Ferrari
tags:
- perl6
- programming
permalink: /:year/:month/:day/:title.html
---
What is the simplest way to use Perl 6? ```rakudobrew``` is an excellent system!

## rakudobrew to the rescue!
-----
If you are a Perl programmer you probably have already used [perlbrew](https://perlbrew.pl/). If you havent' already used ```perlbrew```
go use it, it is a wonderful project that allows you to let several Perl 5 distributions to live on the same machine and, to some extent,
to be used without need to mess with the main (*sysadmin locked*) instance (and it does support also *cperl*)..

[rakudobrew](https://github.com/tadzik/rakudobrew) is the same for the *Rakudo/Perl 6* pair.
With ```rakudobrew``` using a Perl 6 distribution is as easy as:

```shell
% rakudobrew list-available
% rakudobrew build moar 2017.09
```

I have to admit that you need to enjoy a cup of coffee or even a pizza, depending on the speed of your machine, but it works like a charm.
Please note that ```rakudobrew``` supports either *jvm* or *moar* as backend virtual machine, and in the latter case a stable or bleeding edge one.
As for ```perlbrew```, ```rakudobrew``` does support and install a package manager, that up to several months ago was either [panda](https://github.com/tadzik/panda) or [zef](https://github.com/ugexe/zef), but **since [panda has been deprecated](https://github.com/tadzik/panda/commit/cc0df2586c7c0503df4e76374309fb1dc225c544) only zef is supported right now**.

Please note that in the case you want to build for a jvm backend you need a Java compiler in your path or the build will fail almost immediatly.
