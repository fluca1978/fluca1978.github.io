---
layout: post
title:  "Eclipse Keyboard Lag"
author: Luca Ferrari
tags:
- eclipse
permalink: /:year/:month/:day/:title.html
---
How memory can make your Eclipse experience a very nightmare!

# Eclipse Keyboard Lag

I was using Eclipse 2020.03 on Kubuntu Linux, with of course GTK dark theme (if this detail matters) to be homogenous with my Plasma desktop Dark Breeze theme.
<br/>
Editing a Java application was just a mess.
<br/>
There was an abnormal keyboard lag while typing, and in particular the content assistant was not inserting selections if I was hitting any char on the keyboard just after the `Enter` in order to select the choice.
<br/>
The following shows how things were: note the delay between the disappearing of the content assistant (because I selected the proposal) and the appearing of the selected element as text.

<br/>
<br/>
<center>
<img src="/images/posts/eclipse/eclipse_lag.gif" />
</center>
<br/>
<br/>

How to solve?
<br/>
I tried to set an higher activation time for the content assistant, and at first it seemed to work, but after a while the above behavior was resumed. I therefore increased the base memory of my Eclipse in `eclipse.ini` setting:

<br/>
<br/>
```
-Xms512m
-Xmx2048m
```
<br/>
<br/>

and restarted Eclipse.
<br/>
*So far it seems to work!*
