---
layout: post
title:  "Strange Plasma Behaviors (at least to me)"
author: Luca Ferrari
tags:
- kde
- planet-kde-org
- plasma
- linux
permalink: /:year/:month/:day/:title.html
---
A few things I found strange in Plasma.

# Strange Plasma Behaviors (at least to me)

Recently I had to install my computer from scratch, due to the failure of its hard disk. I then installed the most recent of [Kubuntu](https://www.kubuntu.org){:target="_blank"} from scratch, and with that the most available version of [Plasma desktop](https://www.kde.org){:target="_blank"} available for it.
<br/>
Also, due to the COVID-19 scenario, I'm working from home (let's call it *smart working*), and I've prepared a workstation with a double monitor. I've never used Plasma with two monitors before.
<br/>
First of all, I'm running Plasma `5.16.5`, that is not the latest version (that at the time of writing is `5.18.4`), and so part of the things I describe as problems could have been reverted back to what I remember was the old behavior.
<br/>
<br/>
And here comes the strange things...


## A Panel in Another Monitor

It should be trivial: you can move a window from one monitor to the other by right-clicking on the titlebar and move it as you would do with a virtual desktop.


<br/>
<center>
  <img src="/images/posts/plasma/plasma_move_screen.png"
  alt="Move to screen menu" />
</center>

<br/>

However, moving a panel from a screen to the other is not the same. In fact, in order to move a panel you have to *drag it form one screen to the other*. Ok, that could sound trivial, since the same can be done with any window, but placing a menu entry in the panel settings could help people like me that don't like dragging very much. Moreover, having a panel to be created in the screen you are working is surely better than creating it on the main screen.



## Plasma Single Click to Open Folders

When I opened a folder in Dolphin, nothing happened.
<br/>
I clicked the mouse, but nothing. I quickly realized that there was the double click mode active: I had to press double click to open folders, that to me is quite annoying and remembers other operating systems.
<br/>
But hey, this is Plasma, and I know there is a setting for that!
<br/>
I opened the *System Settings* dialog and go straight to the *Input Devices* and then to the *Mouse*, but the setting was not there.
<br/>
I spent several minutes in searching for the setting, and finally I found it under the *Desktop Behavior*:




<br/>
<center>
  <img src="/images/posts/plasma/plasma_single_click.png"
  alt="Desktop Behavior" />
</center>

<br/>
<br/>
I don't want to say this is a poor choice, but there are several forum posts bout this same problem that are pointing to the old location, the *Mouse Options*, that to me seems much more natural.
<br/>
While I understand that this could be a way to uniform the input device click behavior, I also think that different input devices could require different behavior, so I would at least make a clear link between the *Mouse Settings* and the *Deskatop Behavior* to make it clear that the clicking behavior is configured outside the scope of the mouse.
<br/>
That's my opinion.



## Conclusions

Plasma is a great desktop, so far the best I've never far with respect to other operating system (and quite frankly, I've used a lot of them, proprietary or not). However, some choices with regard to configuration and setup are not always *optimal*, considering that here *optimal* is in the eyes of who uses the desktop.
<br/>
I have written this post not to blame Plasma, rather to be useful with my considerations and experience to other users that could find the same settings and behaviors odd to their habits.
