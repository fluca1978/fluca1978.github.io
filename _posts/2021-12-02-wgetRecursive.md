---
layout: post
title:  "wget to recursively download a web site"
author: Luca Ferrari
tags:
- linux
- wget
permalink: /:year/:month/:day/:title.html
---
A simple trick to get a web site totally offline.

# `wget` to recursively download a web site

`wget(1)` is a great tool to download web cotnent from the command line, and it does support also *recursive* downloading, that is it can *follow* links to get more than a single URL content.
<br/>
One problem with the recursive download is that it **does respect the `robots.txt`** file, that instruments automated tools (robots) to not download the content nor inspect the content.
<br/>
Clearly there's a way to instrument `wget` to be *rude* and do what we want:

<br/>
<br/>
```shell
% wget -e robots=off -r '<your-URL>'
```
<br/>
<br/>

The above will turn off the `robots.txt` check, and will download recursively `-r` all the content up to a deep level of 5, that can be tuned to your need.
