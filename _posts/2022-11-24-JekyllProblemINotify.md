---
layout: post
title:  "Jekyll, Bundler and the \"No space left on device - Failed to watch\" error"
author: Luca Ferrari
tags:
- linux
- jekyll
permalink: /:year/:month/:day/:title.html
---
A strange error I found while testing my blog locally

# Jekyll, Bundler and the "No space left on device - Failed to watch" error

Today I started my blog locally, to inspect some posts, and the system refused to start.
<br/>
What did I change since last time? **Nothing!**
<br/>
Apparently, however, I was running the system with an higher load than usual, having several other applications already opened.
Let's face the error:


<br/>
<br/>
```shell
% bundle exec jekyll serve
...
      Generating...
                    done in 12.299 seconds.
jekyll 3.9.2 | Error:  Unable to monitor directories for changes because iNotify max watches exceeded. See https://github.com/guard/listen/blob/master/README.md#increasing-the-amount-of-inotify-watchers .

/var/lib/gems/3.0.0/gems/listen-3.7.1/lib/listen/adapter/linux.rb:32:in `rescue in _configure': Unable to monitor directories for changes because iNotify max watches exceeded. See https://github.com/guard/listen/blob/master/README.md#increasing-the-amount-of-inotify-watchers . (Listen::Error::INotifyMaxWatchesExceeded)
...
/var/lib/gems/3.0.0/gems/rb-inotify-0.10.1/lib/rb-inotify/watcher.rb:74:in `initialize': No space left on device - Failed to watch "/fluca1978.github.io/_site/2019/12/10": The user limit on the total number of inotify watches was reached or the kernel failed to allocate a needed resource. (Errno::ENOSPC)
...
```
<br/>
<br/>

*What the hell is happening?*
<br/>
The first thing that caught my attention was that `No space left on device`: did I run out of disk space? Of course I was not.
Then I have a second look at the error, in particular at the root cause, and followed the [suggested link](https://github.com/guard/listen/blob/master/README.md#increasing-the-amount-of-inotify-watchers){:target="_blank"} to solve the issue.
<br/>
The *sysctl* mentioned there, `fs.inotify.max_user_watches` determines the maximum value of files that the *inotify* subsystem can handle for changes on the filesystem tree.
Once this value has been raised, the Jekyll local site started up smoothly.
