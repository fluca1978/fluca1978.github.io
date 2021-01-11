---
layout: post
title:  "Projectile and switching to a dired buffer or a recently used file"
author: Luca Ferrari
tags:
- emacs
- projectile
- spacemacs
permalink: /:year/:month/:day/:title.html
---
How to switch to a project using a `dired` buffer?

# Projectile and switching to a dired buffer or a recently used file

I like *projectile* because it allows me to quickly switch to a project and kill all related buffers.
However, when you ask to switch to a project, Spacemacs is configured to ask you the buffer you want to open in the target project. Probably this is also true for vanilla Emacs, but I have not checked.
<br/>
Often, what I want to do is to switch to another project and open the dired list of files, so that I can navigate and decide what to open within that project. Why? Because often *I do not remember the structure of the project*!
<br/>
<br()
It simple enough to configure it in your Emacs configuration file:

<br/>
<br/>
```elisp
(setq projectile-switch-project-action 'projectile-dired)
```
<br/>
<br/>
In fact, the `projectile-switch-project-action` can be customied to execute the function `projectile-dired` that in turns open the dired buffer on the project root.

<br/>
But the [Projectile documentation](https://docs.projectile.mx/projectile/configuration.html){:target="_blank"} tells more: it is possible to use the `projectile-commander` that allows for a manual selection of what to do on a project switch, with particular regard to my favourites:
- `D` open a dired buffer in the project root;
- `d` finds a file in the project;
- `e` list recently used files.

