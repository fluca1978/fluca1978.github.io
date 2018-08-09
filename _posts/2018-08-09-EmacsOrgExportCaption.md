---
layout: post
title:  "Emacs Org-mode and the not exported caption problem"
author: Luca Ferrari
tags:
- emacs

permalink: /:year/:month/:day/:title.html
---
Org-mode is an amazing tool I use every day for pretty much a lot of stuff. In these day I've written technical documents in org, but due to a fault of mine, every code snippet was without the appropriate caption. It took me a while to find out what I was missing...


# Emacs Org-mode and the not exported caption problem

I foten have to write technical documentation that includes snippets of code, and thanks to Org-mode this is a piece of cake. My template is as follows:

```shell
#+begin_listing
#+caption: Code showing foo and bar
#+name: listen_4_function
#+INCLUDE: "code/8_04.sql" src sql -i -r
#+end_listing
```

As you can see, I wrap my whole code into a `listing`, with a `caption` (i.e., a title) and a `name` used for cross reference. The code is included via the `#+include` directive, so I'm sure that everytime I export the code I will grab the most updated version out of the version control.

So far, so good. However, since a couple of days, I was not getting the caption out of the export. After a good amount of time, I noted that my listings were all missing a little piece: the `src` directive before the language itself.
As a result, the code was not correctly wrapped, and most notably, the caption was not inserted in the export frame.

Shame on me! I had copied and pasted the listing template over and over, so my whole document was srewed up. Ok, donì't panic, Emacs can solve it with an `M-%` in a bunch of seconds.
<br/>
And happiness came again!


