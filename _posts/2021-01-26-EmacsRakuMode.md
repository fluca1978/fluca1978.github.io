---
layout: post
title:  "Emacs Raku Mode is Amazing!"
author: Luca Ferrari
tags:
- emacs
- raku
permalink: /:year/:month/:day/:title.html
---
A great mode for developing Raku application within Emacs!

# Emacs Raku Mode is Amazing!

I've waiting too long for Emacs, my default and favourite editor, to be able to *help* me in looking at Raku (aka Perl6) code.
<br/>
The good old Perl mode was not suitable, and in fact the following is a look at some Raku code with the Emacs Perl Mode applied:

<br/>
<br/>
<center>
<img src="/images/posts/emacs/raku_mode_1.png" />
</center>
<br/>
<br/>

And here it is the beautiful [Raku Mode](https://github.com/Raku/raku-mode){:target="_blank"} in action on the same piece of code:

<br/>
<br/>
<center>
<img src="/images/posts/emacs/raku_mode_2.png" />
</center>
<br/>
<br/>

The mode also provides a good indentation approach, that allows for a good alignment of Raku elements. As an example, it can turn the below piece of code:

<br/>
<br/>
<center>
<img src="/images/posts/emacs/raku_mode_3.png" />
</center>
<br/>
<br/>

into the following one, sadly it does not trim extra spaces (e.g., after the `:`)

<br/>
<br/>
<center>
<img src="/images/posts/emacs/raku_mode_4.png" />
</center>
<br/>
<br/>




And there is more: the mode provides also bare templates for creating new scripts and modules. Quite frankly, I let the *yasnippets* package to do the work, but this mode is very good.



## Installing into Spacemacs

Since there is no layer available for Spacemacs, at least so far, in order to prevent Spacemacs to delete the package as *orphaned* you need to install it under the `dostpacemacs-additional-packages` variable, in a way similar to the following (the extra packages are not related nor required to Raku mode):

<br/>
<br/>
```lisp
dotspacemacs-additional-packages '(doom-themes
                                      yasnippet-snippets
                                      raku-mode
                                      )
```
<br/>
<br/>
