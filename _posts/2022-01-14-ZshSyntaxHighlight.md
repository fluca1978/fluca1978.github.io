---
layout: post
title:  "Zsh Syntax Highlight"
author: Luca Ferrari
tags:
- shell
- zsh
permalink: /:year/:month/:day/:title.html
---
Zsh has a nice package to provide syntax highlighting on the fly.

# Zsh Syntax Highlight

Across the wide set of features, `zsh` has a nice package I never activated before, even if I was aware of its existance: `zsh-syntax-highlighting`. The package is inspired by *fish shell* and is [available here](https://github.com/zsh-users/zsh-syntax-highlighting){:target="_blank"}.
<br/>
This package provides a *highlight as you type* functionality, so that commands that are mispelled (or not available to you) are shown in red, well written commands and arguments are in green, directories are underlined, and so on.
Installing the package is a matter of seconds, e.g., on OpenBSD:


<br/>
<br/>

``` shell
% doas pkg_add zsh-syntax-highlighting-0.7.1
```
<br/>
<br/>

Then you need to load the plugin **as a last thing in your `.zshrc`:

<br/>
<br/>

``` shell
% tail -n 1  ~/.zshrc
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
<br/>
<br/>

Please make sure the sourced file is in the right place, usually it is on `/usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`, but some package managers could install it on another place.
<br/>
The following is a bare example of the plugin in action (please note that the colored prompt comes from the `adam1` theme).


<br/>
<center>
<img src="/images/posts/zsh/zsh_syntax_highlight.png" />
</center>
<br/>
