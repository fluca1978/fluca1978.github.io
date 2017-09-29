---
layout: post
title:  "Emacs helm"
author: Luca Ferrari
tags:
- emacs
- helm
permalink: /:year/:month/:day/:title.html
---
A quick introduction to **helm** and why I love it.

## Emacs helm
-----

If you are an avid Emacs user you probably already know about [**helm**](https://github.com/emacs-helm/helm), a completion assistant
similar, at glance, to ```ido``` but a lot, lot more powerful.
```helm``` power comes, in my opinion, by two particular properties:
1. the incremental completion via patterns
2. the huge configuration available.

In particular, the *incremental completion* allows for you to place a few characters and see ```helm``` searching for the possible completions for you. The package provides a great flexibility via the configuration parameters, as [well explained here](http://tuhdo.github.io/helm-intro.html).

My personal, minimal, helm configuration so far is the following:

```lisp
(require 'helm-config)
;; activate helm always!
(helm-mode 1)
;; helm complete commands via M-x
(global-set-key (kbd "M-x") 'helm-M-x)
;; frankly, using tab for actions is annoying, change the behaviour
(define-key helm-map (kbd "<tab>") 'helm-execute-persistent-action) ; rebind tab to do persistent action
(define-key helm-map (kbd "C-i") 'helm-execute-persistent-action) ; make TAB works in terminal
(define-key helm-map (kbd "C-z")  'helm-select-action) ; list actions using C-z
;; resize the buffer depending on the number of completions
(helm-autoresize-mode t)
;; helm complete find files, use ack as grep command in find-files
(global-set-key (kbd "C-x C-f") 'helm-find-files)
(setq helm-grep-default-command "ack-grep -Hn --no-group --no-color %e %p %f"
      helm-grep-default-recurse-command "ack-grep -H --no-group --no-color %e %p %f")

					; semantic
(semantic-mode 1)
(setq helm-semantic-fuzzy-match t
      helm-imenu-fuzzy-match    t)

```

As you can see, my configuration is pretty much straightforward. One thing I like the most about ```helm``` is the capability to interact with ```semantic```, that for a developer like me is a huge improvement.
