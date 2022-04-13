---
layout: post
title:  "Formatting SQL code with pgFormatter within Emacs"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- emacs
permalink: /:year/:month/:day/:title.html
---
Editing SQL and PostgreSQL related code within Emacs, in a beautiful war!

# Formatting SQL code with pgFormatter within Emacs

[pgFormatter](https://github.com/darold/pgFormatter){:target="_blank"} is a great Perl 5 tool that parses SQL input and re-format it in a beautiful way.
<br/>
Despite the name, it works with any SQL piece of code, since it does support the standars from *SQL-92* to *SQL-2011*, plus all little keywords and details that are specific to PostgreSQL.
<br/>
<br/>
Being myself an *Emacs addicted*, I reasoned about how to "pkug in" `pgFormatter` into Emacs, and I came up with a short and ugly snippet of code that does the trick.
<br/>
But, being Emacs what it is, there is no particular need to plug in such code, as I will show you in a moment.

## Use `pgFormatter` from Emacs, the *portable* way

Emacs allows users to run a *shell command* over a region or a buffer content. The `M-|` (menomic: *pipe*) does that. With the universal prefix (`C-u`) it can also replace the region or buffer you are running the command against.
<br/>
This means that, given your own Emacs instance, you can format the code within the region by simply doing

<br/>
<br/>
`C-u M-| pg_format`
<br/>
<br/>
where `pg_format` is the name of the executable of pgFormatter (e.g., it is called like that in Rocky Linux).


## A more lispy approach

I [developed a simple and ugly snippet of Lisp](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/emacs/pgformatter.el){:target="_blank"} that can be loaded into Emacs to make the pgFormatter usage quicker.

<br/>
<br/>

``` common-lisp
(defun pgformatter-on-region ()
  "A function to invoke pgFormatter as an external program."
  (interactive)
  (let ((b (if mark-active (min (point) (mark)) (point-min)))
        (e (if mark-active (max (point) (mark)) (point-max)))
        (pgfrm "/usr/bin/pg_format" ) )
    (shell-command-on-region b e pgfrm (current-buffer) 1)) )

```
<br/>
<br/>

The above piece of code defines an interactive function (i.e., a function that can be invoked with `M-x`) named `pgformatter-on-region`. The function defined three variables:
- `b` is the beginning of the region to format;
- `e` is the end of the region to format;
- `pgfrm` is the path to the executable to invoke.

If there is a region active (i.e., `mark-active`) the function operates over the region, otherwise, if no region is applied, it operates on the whole buffer.
<br/>
In the end, the function invokes the shell command `pgfrm` using the internal interactive function `shell-command-on-region` over the current buffer. The last argument, `1` indicates that I want to substitute the content of the current region (or buffer) with the command output.

<br/>
<br/>
In order to execute the formatting, I then just need to `M-x pgformatter-on-region` with either an active region or not. It is also possible to bind the function to a keyboard sequence with something like:

<br/>
<br/>

``` common-lisp
(global-set-key (kbd "C-i") 'pgformatter-on-region)
```
<br/>
<br/>

or a local key map entry.

<br/>
<br/>
The ending result is something like the following:

<br/>
<center>
<img src="/images/posts/emacs/pgformatter.gif" width="50%"/>
</center>
