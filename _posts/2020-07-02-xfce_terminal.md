---
layout: post
title:  "XFCE Terminal cannot execute custom command"
author: Luca Ferrari
tags:
- linux
- xfce
permalink: /:year/:month/:day/:title.html
---
That's why I love Plasma!

# XFCE Terminal cannot execute custom command

Today a friend of mine asked for a very simple task to accomplish in [XFCE](http://xfce.org){:target="_blank"}: *setting a terminal emulator to automatically do an `ssh` connection*.
<br/>
How hard could it be? You configure your terminal to run an external command, for example:

```shell
/bin/bash -c "ssh you@somehost"
```

and that's all! I've done a lot of times in `konsole`, the Plasma terminal emulator.
<br/>
However, XFCE terminal emulator cannot do. Apparently you can set up it:

<br/>
<center>
<img src="/images/posts/xfce/xfce_terminal_1.png" />
</center>
<br/>

but what happens behind the hood is that the command is translated into a single string instead of a command with arguments, then the command is searched as an executable on disk and, of course, not being found, it is not run:

<br/>
<center>
<img src="/images/posts/xfce/xfce_terminal_2.png" />
</center>
<br/>

I've tried any human-available string escape, but nothing works, **so the solution is to wrap your command(s) into a script without arguments and call it as a *custom comamnd*! **
<br/>
It is fun that the [official documentation does not mention this problem, and most notably does not mention the setting at all](https://docs.xfce.org/apps/terminal/preferences){:target="_blank"}!
