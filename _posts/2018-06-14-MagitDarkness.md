---
layout: post
title:  "Magit: in the beginning there was the darkness"
author: Luca Ferrari
tags:
- git
- emacs
permalink: /:year/:month/:day/:title.html
---
Magit is really my favorite porcelain for git, and I've just discovered a fun message in it...

# In the beginning there was the darkness

Usually I use *Magit* over an existing repository, never on a *freshly created one*.
What happens when you have a new repository? `git` does not report anything, obviously:

```sh
% git status
On branch master

Initial commit

nothing to commit (create/copy files and use "git add" to track)


% git log
fatal: your current branch 'master' does not have any commits yet
```

but what about the very situation in `magit`?
Here's a screenshot for the above repository:

![magit_initial_repository](/images/posts/magit_initial_repository.png)

*How fun!*
