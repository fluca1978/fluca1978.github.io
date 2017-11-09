---
layout: post
title:  "Magit vs EGit"
author: Luca Ferrari
tags:
- magit
- git
- egit
permalink: /:year/:month/:day/:title.html
---
I don't know if I hate *EGit* the much or love *Magit* so much!

## Magit vs EGit
-----

My day-by-day job is performed mainly within two enviroments: *Eclipse* for all Java based development and *Emacs* for all the rest (including also not-developing!). Since I tend to keep track of several projects using *Git* whenever possible, I need to integrate both environments with git, and in the former case [EGit](http://www.eclipse.org/egit/) is the solution, while in the latter [Magit](https://magit.vc/) is the tool.

The following are just a bunch of personal considerations, I do not intend in any case to offend anyone of the above projects and their developers. I'm sure there are smart developers behind both, but comparing the two tools lead me think one is better than the other.


*[EGit](http://www.eclipse.org/egit/) is the worst!*
Sorry to be harsh, but that's what I really think.


What is wrong with EGit?

### The commit
---
The *Git staging* panel is horrible, and I don't see the point in having a three way view showing me the staging area and the unstaged files. Here [Magit](https://magit.vc/) has a cleaner approach showing me a pre-compiled commit message in the pure *git style* with the list of the staging area at the bottom. Why is that clearer? Because I can concentrate on the commit message, and not on other dirty information.

### Availability
---

In EGit some git commands are hidden depending on the context. For instance the *branching* is only available when working at the project level, not at the file level. In Magit you open the *status* buffer, and that's the entry point to the whole git system and commands, so you can always do branching stuff starting from a single file.

### Renaming and Organizing Git Commands
---

EGit does a strong renaming of things, and that would be good if git was a pretty damn unknown thing, or if the EGit UI was proposing the same structure of other SCM (e.g., SVN), instead it is just providing an interface from scratch.

As an example, the *checkout* operation is within the *Switch To* menu.
The *Push*, *Pull* and *Merge* operations are not renamed, but then a strange *Advanced* menu contains some entries for other branching operations. Tagging is performed via the above *Advanced* menu, so there's a little confusion here: what the hell is *advanced* supposed to be?
I understand, but not believe, that some special branching operations like renaming could live on their own sub-menu, but tagging is a quite simple and routinely performed task, so why should be considered an advanced feature?

Then there's a *Remote* menu entry that contains the same *Push* and *Pull* operations. Of course, the latter will execute against a remote repository/branch, while the former against the local repo, but is really there the need to keep things split into two levels? We are talking to developers here, it is supposed a developer knows when/where to *push* and *pull*.

So what does Magit about the above? First of all it does not rename commands, so what you are going to expect to work in git will work also in Magit and the other way around. Moreover, when you perform a *push* or *pull* operation Magit will prompt you for "where" to perform the action against. The choice is guided, so there's no extra brain power but following the on-screen instructions and suggestions.


### Conclusions
---

I really don't know if I love Magit the most or hate EGit with a passion.
**But I'm really sure that while working with Magit will not break my workflow with git commands, leaving me able to perform command line commands with the very same name, the other way around will result in me searching for a `switch to` command for performing checkouts and similar actions.**
