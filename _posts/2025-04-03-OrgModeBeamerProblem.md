---
layout: post
title:  "The strange problem of Org-Mode Beamer export when using a standout frame"
author: Luca Ferrari
tags:
- emacs
- latex
permalink: /:year/:month/:day/:title.html
---
A strange problem I encountered while preparing a presentation...

# The strange problem of Org-Mode Beamer export when using a standout frame

I use Emacs Org-Mode for pretty much all documents I produce, including of course Beamer presentations.

Today I was preparing the presentation for the [PgTraining OpenDay 2025](https://pgtraining.gitlab.io/openday/){:target="_blank"}, clearly using Org-Mode and the Beamer export.

I had, in my slide template, the following slide:

```
* Questo talk
:PROPERTIES:
:BEAMER_OPT: standout
:END:

#+begin_center
#+latex: {\HUGE \textcolor{yellow}{ \textbf{
pgagroal
#+latex: } } }
#+end_center
```


Nothing really strange, a **standout** slide with a centered yellow title.

While working on the slides, I changed the previous to the following:


```
* Questo talk
:PROPERTIES:
:BEAMER_OPT: standout
:END:

#+begin_center
#+latex: {\HUGE \textcolor{yellow}{ \textbf{
~pgagroal~
#+latex: } } }
#+end_center
```

that was translated, correctly (according to me), to the following:


```
\begin{frame}[label={sec:org6be9b73},fragile,standout]{Questo talk}
 \begin{center}
{\HUGE \textcolor{yellow}{ \textbf{
\texttt{pgagroal}
} } }
\end{center}


\end{frame}
```

**The issue was that, all the slides after the previous one, kept the inverted colors of the handout slide, so the text was white on a white background, becoming immediatly invisible**.

I don't understand why LaTeX and Beamer behave like that, but I had to remove the code formatting from the text.
