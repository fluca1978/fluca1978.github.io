---
layout: post
title:  "Jekyll Simple Stat has grown to Raku!"
author: Luca Ferrari
tags:
- raku
- perl5
- blog
permalink: /:year/:month/:day/:title.html
---
My Perl 5 script to generate statistical data about a Jekyll blog has been rewritten in Perl 6 (or, ehm, Raku)!

# Jekyll Simple Stat has grown to Raku!

During the [New Year's holiday](https://fluca1978.github.io/2020/01/05/NewStats.html){:target="_blank"}, I made [different changes to my very simple Perl 5 script](https://github.com/fluca1978/jekyll-simple-stats/blob/master/jekyll_simple_stats.pl){:target="_blank"} that I use to generate statistical data about my blog.
<br/>
While doing such changes, I thought about how to implement a single year generation, because after all you should not re-generate every time all the blog stats. I ended up to the conclusion that the current data structure in my script was too simple to easily process a single year, and therefore some more complex object was required.
<br/>
Now, since with the new year I started again reading the excellent brian d foy books on Perl 6 (that I read on December 2018), I decided to try to implement it in Raku. After all, how long could it take?
<br/>
<br/>
I had to work over for a few days, since it was not clear which classes have to do what, and effectively I changed the internal data structure a couple of times; this is a normal process if you start from scratch and it is the beauty of working with a dynamic language!
<br/>
<br/>
And so it comes today: **my [Raku version is now merged and available](https://github.com/fluca1978/jekyll-simple-stats/blob/master/jss.p6){:target="_blank"}!**
<br/>
<br/>
This new version has several new features and drawbacks, so more in general is *different in concept and implementation* from the old one. **The Perl 5 version of this application will be no more mantained**, or better, it could be I will have time to rewrite it to be consistent with the Raku version, but I don't think this will happen in the near future.

## Features of the Raku Version

The new version is much more consistent in the file naming, and in particular tends to store all the resulting files and images into a subdirectory *stats*. 
<br/>
The program file name itself is much shorter, going from `jekyll_simple_stats.pl` to `jss.p6`.
<br/>


The application accepts only one mandatory argument, that is the *home folder* of the blog, so that all other directories like the images one, the include ones and so on, are automatically computed by the script (let's call it as *convention over configuration*).
<br/>
<br/>
There is another important parameter that allows you to optionally indicate the year you want to generate. This is useful to make the whole generation a lot easier, since you probably don't want to modify every time the stats about past years.
<br/>
In particular, the `--year` parameter accepts either a numberic year or a special keyword like:
- `current` to generate only the current year (as from the system clock);
- `previous` or `last` to generate only the previous here (useful across the year boundary).
<br/>
<br/>
Moreover, the program provides a quite extensive online help that can be accessed via the `--help` parameter and has been implemeted by means of a *multi MAIN* entry point.


## Possible Future Development

In the future, it could make sense to remove the hard-coded GNUPlot templates from the script and use some templating system to generate them.
<br/>
Another possible enhancement would be a parallel execution, with one Promise to execute a single year.
<br/>
<br/>
Let's see what comes to mind (and how many time do I have).
