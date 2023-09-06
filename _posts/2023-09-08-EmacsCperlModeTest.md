---
layout: post
title:  "Using CPerl Mode in Emacs when writing test files"
author: Luca Ferrari
tags:
- emacs
- perl
permalink: /:year/:month/:day/:title.html
---
A simple configuration setting that enables CPerl-Mode when editing test files.

# Using CPerl Mode in Emacs when writing test files

When developing in Perl using Emacs, I usually adopt the `cperl-mode` major mode, that works great for my needs.
However, this mode is not enabled by default when editing test files, that is those with the `.t` extension.
<br/>
Luckily, Emacs is totally customizable, and therefore it does suffice to add something the following line in the startup configuration file to get the trick:

<br/>
<br/>
```lisp
(add-to-list 'auto-mode-alist '("\\.t\\'" . cperl-mode))

```
<br/>
<br/>


and, after the evaluation, every time Emacs will visit a test file with `.t` extension the CPerl Mode will be automatically loaded.
