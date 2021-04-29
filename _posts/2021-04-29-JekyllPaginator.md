---
layout: post
title:  "Jekyll and the paginator gem"
author: Luca Ferrari
tags:
- jekyll
- linux
permalink: /:year/:month/:day/:title.html
---
Some trouble to get my local copy of the Jekyll blog to run.

# Jekyll and the paginator gem

I've recently changed my main computer, and this resulted in the need to reinstall a lot of software, including Jekyll to run locally my blog.
<br/>
However, I was unable to start the usual `bundle exec jekyll serve` command due to a traceback related to missing dependencies. The first fix was to include the `jekyll` Gem in the `Gemfile` file:

<br/>
<br/>
```shell
gem "jekyll"
```
<br/>
<br/>

Then I was able to start the local site, but all my posts have disappeared. Or better, they were there in the `_site_` directory, but they were not shown at all. I was able to directly render everyon with a specific link, but not to gain access to the list.
I noticed that the command was reporting a warning at startup:

<br/>
<br/>
```shell
% bundle exec jekyll serve
Configuration file: /home/luca/git/fluca1978.github.io/_config.yml
       Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `plugins: [jekyll-paginate]` in your configuration file.
            Source: /home/luca/git/fluca1978.github.io


```
<br/>
<br/>

The solution was to add the `plugins` line into the `.config.yml` file, and to install it as a Gem.
So, as first step I modified the `Gemfile` adding the paginator plugin:

<br/>
<br/>
```shell
gem "jekyll-paginator"

```
<br/>
<br/>

and then I installed using `bundle`:

<br/>
<br/>
```shell
% bundle install
```
<br/>
<br/>

Then I modified the `.config.yml` file to include the plugins, please note the name of the paginator that is different from the Gem:

<br/>
<br/>
```shell
plugins: [jekyll-paginator]
```
<br/>
<br/>

and I was able to start my site again.
