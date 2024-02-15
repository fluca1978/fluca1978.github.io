---
layout: post
title:  "Emacs, Tramp and (remote) zsh"
author: Luca Ferrari
tags:
- emacs
- zsh
permalink: /:year/:month/:day/:title.html
---
A simple solution to a problem I face often.

# Emacs, Tramp and (remote) zsh

I often use Emacs Tramp as a way to remotely connect to other machines while editing with my local Emacs configuration.
Essentially, Tramp opens an ssh connection to the remote host allowing me to edit files and moving within the filesystem thanks to `dired`.

I prefer to use **zsh** as my default shell, and sometimes I forget to configure properly my shell to get Tramp working. It happens, in fact, that when connecting to `zsh` Tramp hangs.

The solution is quite simple and [is reported in the Tramp manual](https://www.gnu.org/software/emacs/manual/html_node/tramp/Remote-shell-setup.html#Other-remote-shell-setup-hints){:target="_blank"}: add to the `.zshrc` on the remote machine a line that prevents the *ZLE* to fire up when Tramp is opening a connection.
The following is the piece of code to place in `.zshrc`:

<br/>
<br/>
<center>
```shell
[[ $TERM == "dumb" ]] && unsetopt zle && PS1='$ ' && return
```
</center>
<br/>
<br/>

The idea is quite simple: when connecting Tramp sets the `TERM` variable to `dumb`, therefore if the terminal is found to be a `dumb` one the `zle` is disabled and prompt is changed to something similar to the Bash shell (i.e., `$`).
