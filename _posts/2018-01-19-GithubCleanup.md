---
layout: post
title:  "GitHub clean-up time!"
author: Luca Ferrari
tags:
- git
- github
permalink: /:year/:month/:day/:title.html
---
I decided to cut down the number of repositories I have on my [GitHub account](https://github.com/fluca1978?tab=repositories).

# Remove old code, keep it clean!

I spent some time removing old repositories on my [GitHub account](https://github.com/fluca1978?tab=repositories).

Why?

Because as a developer I believe I must keep things organized and clean.
And even if I'm not paying a cent for the hosting of such code, **I don't want to screw up people first impression bloating my repository number**.

Therefore I decided to keep only repositories I do really contribute to, removing also those I today participate directly.

That's it, my account deflated from around 30 repositories to **13, the half!**
That's are the only repositories I do care about today, the number will grow again, but as today that's it.

What about all the other crappy code that I tried to merge into upstream repos and was laying around on my account?
It is simply gone! If the code has not been merged within a few years, chances are it will never be merged. And most notably, I don't want to
keep around examples of bad code, they are useless even to me (the author).



## Re-organize my local git repositories

Now, the above gave me a push in re-organizing my whole local repositories.
I decided to follow these simple, or better, *trivial* rules:
1. all repositories go to a `~/git` folder, so I don't have to search for them across the disk;
2. all repositories I do contribute actively got on the first level of such directory, other repositories I simply track or use as inspiration go into a `misc` subfolder, so that I can keep a clear layout;
3. all my remotes have been renamed to `github`, and that has always been. I do not like the name `origin`, it's that simple. If the repository comes from another service, the remote is renamed accordingly, so that I can always keep an idea of where the real code is without having to parse the pull/push URLs;
4. all tracking branches to the original repository get the remote name of `upstream`, so I know that when I switch to `upstream` I'm in the danger public zone;
5. if the repository is not mine, but I'm one of the committer, the only remote is `upstream` just to remind me I'm always in the danger zone!
