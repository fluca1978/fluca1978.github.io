---
layout: post
title:  "No fonts in Emacs (on Ubuntu) ?"
author: Luca Ferrari
tags:
- emacs
- linux
permalink: /:year/:month/:day/:title.html
---
An annoying problem with a fresh compiled Emacs...

# No fonts in Emacs (on Ubuntu) ?

A few days ago I downloaded and installed Emacs 28.1, and then I **compiled** it on my fresh Kubuntu Neon system.
<br/>
When I launched my fresh Emacs, I had a surprise: *the font system was not correctly initialized and the only **default font** I could choose from was between `courier`, `misc` and `family`.*
<br/>
I started working with the system font cache `fc-cache` and related commands in the hope to fix the problem. But nothing worked.
<br/>
The packaged Emacs was working fine, but it was too old since Kubuntu installed Emacs 26!
<br/>
<br/>
After a few days, the solution came from the [emacs mailing list](https://lists.gnu.org/archive/html/help-gnu-emacs/2022-04/msg00235.html){:target="_blank"}: **I was missing the GTK3 development libraries** and therefore when compiling Emacs it was failing back to the default fontset.
<br/>
**The solution therefore was as simple as `sudo apt install libgtk-3-dev` and issue an Emacs compile cycle.**
