---
layout: post
title:  "Jekyll Simple Stats exploits Chart::Gnuplot"
author: Luca Ferrari
tags:
- raku
- blog
permalink: /:year/:month/:day/:title.html
---
Another little improvement to my personal statistical data analyzer for Jekyll blogs.

# Jekyll Simple Stats exploits Chart::Gnuplot

My own Raku [application to create simple stats about how much do I blog](https://github.com/fluca1978/jekyll-simple-stats){:target="_blank"} on my Jekyll platform, named *Jekyll Simple Stats*, and implemented in Raku (aka Perl6), has grown a little more.
<br/>
<br/>
Since its inception, the script was a kind of giant wrapper to feed [Gnuplot](http://www.gnuplot.info/){:target="_blank"} to genereta some raw histograms. In order to achieve that, I was producing during the process a few *Comma Separated Values (CVS)* and Gnuplot scripts to generate the images.
<br/>
<br/>
A few days ago I discovered [Graph::Gnuplot](https://github.com/titsuki/raku-Chart-Gnuplot){:target="_blank"}, a module for Raku that handles all the machinery to make Gnuplot working without having to manually generate the data and the scripts. [After a few problems with the installation](https://github.com/titsuki/raku-Chart-Gnuplot/issues/41){:target="_blank"** and a few difficulties in understanding how to produce the in-memory dataset, I was able to get it running from my application.
<br/>
<br/>
**The result was much more professional than the one I obtained by my scripts** and, moreover, I don't have to handle temporary files by myself.
<br/>
<br/>
And thanks to this much more integrated graph system, adding an option to my application to customize the color of the graphs was really easy!
