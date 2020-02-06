---
layout: post
title:  "Git Rename Limit"
author: Luca Ferrari
tags:
- git
- magit
- linux
permalink: /:year/:month/:day/:title.html
---
I saw a new (to me) warning in Magit...

# Git Rename Limit

I was working on a quite large project, with hundreds of files, and one of the branch diverged very much from the most *stable* working copy.
<br/>
When I checked out that "old" branch, Magit presented me a strange warning, that was of course coming from the underlying Git:

```
% git merge LANG_PHP_7
...
Performing inexact rename detection
...
warning: inexact rename detection was skipped due to too many files.
warning: you may want to set your merge.renamelimit variable to at least 3362 and retry the command.
```

What is that?
<br/>
Digging the documentation, I found that even if Git has the automatic rename detection (option `diff.renames`), it gives up when the number of files to *detect as renamed* is higher than the parameter `diff.renameLimit`. However, since I was *merging*, the options related to the values are `merge.renames` and `merge.renameLimit` that defaults to their `diff` counterparts.

<br/>
<br/>
The warning is not a problem related to the content, that means Git is not going to loose anything (of course!), rather it will no try to detect automatically files that has been renamed or moved.
<bR/>
[Here you can find a quick rationale](https://git.vger.kernel.narkive.com/6FCkNGgE/warning-too-many-files-skipping-inexact-rename-detection){:target="_blank"} about the message, but the important thing to know is that *this feature was a part of Git since a long time, even if the program was not telling it to the users!*
