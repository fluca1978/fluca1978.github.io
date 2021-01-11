---
layout: post
title:  "uninitialized constant Jekyll::Filters::DateFilters"
author: Luca Ferrari
tags:
- ruby
- jekyll
permalink: /:year/:month/:day/:title.html
---
After a system update I was no more able to run my Jekyll instance...

uninitialized constant Jekyll::Filters::DateFilters
---
After a massive update on my personal computer, I was no more able to run the local instance of this blog.
In particular, there was an error related to Jekyll, as shown below:


<br/>
<br/>
```shell
% bundle exec jekyll serve
Traceback (most recent call last):
        9: from /usr/local/bin/jekyll:23:in `<main>'
        8: from /usr/local/bin/jekyll:23:in `load'
        7: from /var/lib/gems/2.7.0/gems/jekyll-3.4.5/exe/jekyll:6:in `<top (required)>'
        6: from /var/lib/gems/2.7.0/gems/jekyll-3.4.5/exe/jekyll:6:in `require'
        5: from /var/lib/gems/2.7.0/gems/jekyll-3.4.5/lib/jekyll.rb:36:in `<top (required)>'
        4: from /var/lib/gems/2.7.0/gems/jekyll-3.4.5/lib/jekyll.rb:82:in `<module:Jekyll>'
        3: from /var/lib/gems/2.7.0/gems/jekyll-3.4.5/lib/jekyll.rb:82:in `require'
        2: from /usr/lib/ruby/vendor_ruby/jekyll/filters.rb:5:in `<top (required)>'
        1: from /usr/lib/ruby/vendor_ruby/jekyll/filters.rb:6:in `<module:Jekyll>'
/usr/lib/ruby/vendor_ruby/jekyll/filters.rb:9:in `<module:Filters>': uninitialized constant Jekyll::Filters::DateFilters (NameError)
Did you mean?  DateTime
```
<br/>
<br/>


I therefore decide to remove the packaged version of Jekyll (that was `3.1-something` with regard to `3.9` currently available) and also Ruby, so to start over a very clean environment:


<br/>
<br/>
```shell
% apt remove jekyll
% apt remove ruby
% apt install ruby
% sudo gem install bundler
```
<br/>
<br/>


Then, of course, I installed all the dependencies for the static site content:

<br/>
<br/>
```
% bundle install
% bundle update
```
<br/>
<br/>

and finally I was able to serve this blog on my local machine again!
<br/>
It happens to me more and more ofter: (K)Ubuntu is messing up my environment and is not able to handle updates in a proper way.
I was using Kubuntu 20.04, and I have updated all my stuff quite regularly but in the last months I have been away from the keyboard, and this could have been produced an invalid *all-in-one* update.
<br/>
Or maybe it could be the update done via `Discover`.
<br/>
Or I don't know...
