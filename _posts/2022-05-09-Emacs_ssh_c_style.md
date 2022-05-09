---
layout: post
title:  "Emacs: formatting C code over SSH"
author: Luca Ferrari
tags:
- emacs
- c
permalink: /:year/:month/:day/:title.html
---
Editing remote files with Emacs is usually an easy task, thanks to *TRAMP*. But what about code style?

# Emacs: formatting C code over SSH

I do quite often edit remote files with Emacs by means of *TRAMP*, that means *over SSH*.
<br/>
It works simply great!
<br/>
<br/>
However, a problem I found while editing some *strict-style* project written in C, is about instrumenting Emacs to observe a *per-project* `c-style` configuration. **`.dir-locals.el`** files can rescue you, but the problem is about their scope: such file are local only,
<br/>
Or that is what I thought!
<br/>
*TRAMP* has a special variable named `enable-remote-dir-locals` that can be set to a non-`nil` value to make Emacs aware of `.dir-locals.el` files on the remote side.
<br/>
Therefore, on the remote machine, I've:

<br/>
<br/>

``` shell
$ cat .dir-locals.el
;; Directory Local Variables
;;; For more information see (info "(emacs) Directory Variables")

((c-mode . ((c-file-style . "ellemtel")
            (indent-tabs-mode . nil))))
```
<br/>
<br/>

and on my local Emacs instance I placed:

<br/>
<br/>

``` emacs-lisp
(setq enable-remote-dir-locals t)
```
<br/>
<br/>

and that's all I need!
<br/>
See the [Emacs documentation about dir-locals](https://www.emacswiki.org/emacs/DirectoryVariables#h5o-4){:target="_blank"} for more information.
