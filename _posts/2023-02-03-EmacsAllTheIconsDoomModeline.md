---
layout: post
title:  "Emacs Doom Modeline and Icons Scrambled"
author: Luca Ferrari
tags:
- emacs
permalink: /:year/:month/:day/:title.html
---
A quick fix that I tend to forgot when using Doom Modeline.

# Emacs Doom Modeline and Icons Scrambled

Since a year or so I switched to using the [Doom Modeline](https://seagle0128.github.io/doom-modeline/){:target="_blank"}.
When I install a fresh Emacs, I activate the `doom-modeline` and also `all-the-icons` but I often forget to install the right fonts to display the icons.
Therefore, the initial result is as follows:

<br/>
<br/>
<center>
<img src="/images/posts/emacs/all_the_icons_1.png" />
<center>
<br/>
<br/>

The trick to fix this is to **manually** run the command **`M-x all-the-icons-install-fonts`**, after that Emacs will display the correct icons in the modeline:


<br/>
<br/>
<center>
<img src="/images/posts/emacs/all_the_icons_2.png" />
<center>
<br/>
<br/>
