*---
layout: post
title:  "Emacs Projectile Tricks: switching between files"
author: Luca Ferrari
tags:
- emacs
- projectile
- development
permalink: /:year/:month/:day/:title.html
---
A couple of nice trickes about Projectile and the management of files.

# Emacs Projectile Tricks: switching between files

In the last year, I took back development on **C language** because I've been involved in [pgagroal](https://agroal.github.io/pgagroal/){:target="_blank"}, a fast connection pooler for PostgreSQL.

My development environment for this (and a lot more) project has been **Emacs**, and it has been a good way to learn more about [Projectile](https://projectile.mx/){:target="_blank"}, a great tool to manage project source trees within your favourite editor.

One interesting feature, that is saving me a lot of time, and that is not so important in other programming languages, is the switching between an implementation file and an header file. Well, Projectile is *smart enough* that if you run `M-x projectile-find-other-file` (or a variant of that command), the system will switch *between a source file `.c` to its counterpart header `.h` file and viceversa*.
The command, in my setup, is bound to **`C-c p a`**.
For this stuff to work, the two files must have the same name, but it does not matter if they are or not within the same directory.


Another useful feature, especially when working with a project not on daily basis, is `M-x projectile-find-file-dwim`, that in my configuration is mapped to **`C-c p g`**: this command searches for a file in the project trying to guess what you are looking for. This means that it is not mandatory to remember the tree structure to find a file!

Last but not least, the super `M-x projectile-find-tag`, usually known as **jump** action, in my configuration bound to `C-c p j`: it jumps to the definition of the current word at point (e.g., a function name) or asks for the definition to search for.
