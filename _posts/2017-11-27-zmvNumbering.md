---
layout: post
title:  "zmv naming files"
author: Luca Ferrari
tags:
- zmv
- shell

permalink: /:year/:month/:day/:title.html
---
A quick and fast way to number all files with `zmv`,

# zmv naming files
-----

As well known, `zmv` is one of the goodies that come for free when using the great [zsh](http://www.zsh.org/) shell, and it allows for a fast and easy way to bulk renaming files using a pattern like syntax. As of today, I did not need to number renamed files, but of course the `zmv` provides a quick way to do it. In fact, `zmv` allows for shell interpolation within the command itself, so that for instance:

```shell
% counter=1 zmv '*.jpg' 'foo-$((counter++)).jpg'
```

does the trick! The idea is simple: the `counter` variable is incremented each tiem `zmv` requires a new evaluation of the name, and this is done via the `$(( ))` arithmetic expansion.
