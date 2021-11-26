---
layout: post
title:  "Configuring Emacs using Org-mode: org-babel"
author: Luca Ferrari
tags:
- emacs
- org-mode
permalink: /:year/:month/:day/:title.html
---
A glance at a well-known way to keep your configuration under control.

# Configuring Emacs using Org-mode: org-babel

Emacs Org-mode is a great tool, and includes support for the so called /literate programming/. In short, you write your documentation at first, and within the documentation you insert snippets of code that can be /run/ (executed, compiled) by Emacs itself in order to produce a /concrete result/.
<br/>
And of course, **Emacs can use itself Org-mode to *auto-configure* the experience**!
<br/>
The trick is to use a package called **org-babel** and start to write your configuration into an Org-mode file.

# Quick, show me the code!

There are plenty of tutorials online about this subject, so I will not go into deep details here. However, there are two main actions to take in order to configure Emacs via Org-mode files:
- configure a startup file to load `org-babel` and make it parse one file (or more) written in Org-Mode;
- create an Org-mode file with `emacs-lisp` block of codes that will do the configuration.

## Step 1: `init.el`

Edit the startup file `init.el` (under `~/.emacs.d/init.el`) and place the following piece of code:


<br/>
<br/>
```lisp
(require 'org)
(org-babel-load-file
 (expand-file-name "emacs.org"
                   user-emacs-directory))

```
<br/>
<br/>

The above snippet of code will load a file named `emacs.org`, that must be in the same directory of the `init.el` one, and will parse it. In particular, `user-emacs-directory` tells to the `expand-file-name` function to use the `~/.eamcs.d` folder, so that the Org-mode file will be `~/.emacs.d/emacs.org`.


## Step 2: write `emacs.org`

Now, write a *usual* Org-mode file named `~/.emacs.d/emacs.org` and put your text, your comments and the block of codes that you need to configure Emacs. **The code blocks must be of type `emacs-lisp`**.
<br/>
As a bare example:


<br/>
<br/>
```
#+title: Emacs Configurtion File
#+author: Luca Ferrari


* Packages
** Configure packages

#+BEGIN_SRC emacs-lisp
      ;; Initialize package sources
      (require 'package)

      (setq package-archives '(("melpa" . "https://melpa.org/packages/")
                               ("melpa-stable" . "https://stable.melpa.org/packages/")
                               ("org" . "https://orgmode.org/elpa/")
                               ("elpa" . "https://elpa.gnu.org/packages/")))

      (package-initialize)



    (unless (package-installed-p 'use-package)
        (package-refresh-contents)
        (package-install 'use-package) )


  (setq use-package-verbose t)
  (setq use-package-always-ensure t)
    (require 'use-package)
 #+END_SRC

 
* User Interface Customization
** Menu, Tool and Scroll Bars
All the bars have a /mode/ toggle that can be set to ~-1~ to indicate it has to be deactivated.
The ~menu-bar-mode~ considers the top menu bar, while ~tool-bar-mode~ is for the graphical tool bar.
The ~scroll-bar-mode~ is for the lateral scroll bar, but unlike the ~tool-bar-mode~, disabling the scroll bar in a non graphical environment causes a startup.
For this reason, the scroll and tool bars are deactivated only if ~display-graphic-p~ is non ~nil~, to indicate the system is running with a graphical display.

The ~progn~ function allows the sequential evaluation of its body and returns the last evaluated expression.

#+BEGIN_SRC emacs-lisp
(menu-bar-mode -1)
(if (display-graphic-p)
    (progn
       (tool-bar-mode -1)
       (scroll-bar-mode -1) ) )
#+END_SRC


** Highlight Current Line
Activate highlight line mode globally, and set a background color.
Setting the foreground color to ~nil~ prevents the line to loose the syntax hightlight.
#+begin_src emacs-lisp
  (global-hl-line-mode 1)
  (set-face-background 'hl-line "#313131")
  (set-face-foreground 'highlight nil)
#+end_src

   
** Guru Mode
Let's use the Emacs keybindings to move!
#+begin_src emacs-lisp
  (use-package guru-mode
  :ensure t )
(setq guru-warn-only 1 )
#+end_src


```
<br/>
<br/>


As you can see the initial part of the file is the preamble (optional) of an Org-mode document.
<br/>
The very first part is the installation and load of the `use-package**, that is the system I use to automatically install packages.
<br/>
**It is important you configure your package manager at the very beginning, so to use it later on in the rest of the document file**, and I included it in the Org-mode so that I can easily drop such file on a fresh machine and get Emacs do all the stuff for me!


# Conclusions

Emacs Org-mode based configuration is simple, thanks to `org-babel`, and can be a very interesting way to craft and share your own configuration in a *less lispy mode* and with a *more human readable* approach!
