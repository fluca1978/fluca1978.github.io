---
layout: post
title:  "Using JetBrains Fonts in Emacs"
author: Luca Ferrari
tags:
- emacs
permalink: /:year/:month/:day/:title.html
---
Make your text editor more appealing.

# Using JetBrains Fonts in Emacs

I live within Emacs, and my setup is very minimal and distraction free.
However, the monospace fonts used in my configuration were not so good, and the following is an example of the everywhere present `Noto Mono` font:

<center>
<br/>
<img src="/images/posts/emacs/jetbrains_fonts_1.png" alt="Emacs with Noto Mono font" />
<br/>
</center>

I discovered the very beautiful [JetBrains Monospace fonts](https://www.jetbrains.com/lp/mono/){:target="_blank"} and decided to give them a try.

This is how it appears on the very same snippet of Perl code:

<center>
<br/>
<img src="/images/posts/emacs/jetbrains_fonts_2.png" alt="Emacs with JetBrains Mono fonts" />
<br/>
</center>

The procedure to install the fonts is quite straightforward:
- download the fonts from [JetBrains](https://www.jetbrains.com/lp/mono/){:target="_blank"};
- extract the archive
- copy the `.ttf` fonts into `$HOME/.local/share/fonts`;
- run `fc-cache` to update the cache.

Then, on the Emacs side, activate the fonts with something like:

<br/>
<br/>
```lisp
(set-face-attribute 'default nil :font "JetBrains Mono" :height 200  )
```
<br/>
<br/>

Note that I increased the size of the font, since it appears to me a little shorter than the Noto one.
