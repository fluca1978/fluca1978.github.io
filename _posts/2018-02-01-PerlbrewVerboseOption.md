---
layout: post
title:  "Perlbrew: added the verbose option"
author: Luca Ferrari
tags:
- perl
- perlbrew
permalink: /:year/:month/:day/:title.html
---
Another little contribution to the great `perlbrew` application.

# Perlbrew `verbose` options for both `available` and `list`

A lot of time ago I submitted a patch to `[perlbrew](http://perlbrew.pl)` that aumented the output of the commands `list` and `available` in order to provide much more information to the user. For instance, the `list` command shows also when a Perl version has been isntalled.


A few days ago I was thrown over an [issue](https://github.com/gugod/App-perlbrew/issues/597) about how such output verbosity can break existing scripts.

Therefore, [I decided to implement](https://github.com/gugod/App-perlbrew/pull/598) the `verbose` option that, once enabled, will show the extra information, otherwise leaving them out and allowing for a much easier parsing of the command output by other applications.
Now, it is quite clear that, as discussed [in the issue](https://github.com/gugod/App-perlbrew/issues/597) this is a clear warning that it should be better to implement a way to keep a stable output while improving it. In other words, **I do believe it is time to implement a kind of JSON (or alike) stable output leaving out the normal text be free to change by every release**.
