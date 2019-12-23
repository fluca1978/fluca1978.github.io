---
layout: post
title:  "Windows on Rails (blue screen of death on train)"
author: Luca Ferrari
tags:
- microsoft


permalink: /:year/:month/:day/:title.html
---
At least the train has rails to avoid a crash...

# Windows on Rails (blue screen of death on train)

Last saturday I took the local train, named *Gigetto*, that pretty much every day drives me to Modena.
Of course, my attention was immediatly catched by a strange blue screen on an internal wall:
<br/>
<br/>
<center>
<img src="/images/posts/gigetto/blue_screen_1.jpg" width="30%" />
</center>
<br/>
<br/>
<br/>
Getting closed to the monitor, it revealed the well known *Microsoft Windows Blue Screen of Death*, and in particular the screen was turned 180 degrees, so it was not possible to read it:

<br/>
<br/>
<center>
<img src="/images/posts/gigetto/blue_screen_2.jpg" width="50%" />
</center>
<br/>
<br/>
<br/>

Rotating the image, it appears clear that there is a problem with a `DLL`:
<br/>
<br/>
<center>
<img src="/images/posts/gigetto/blue_screen_3.jpg" width="50%" />
</center>
<br/>
<br/>
The responsible DLL is `iegd3dga.dll` which is an Intel Embedded Graphic Driver, that means the device has a problem with the graphic card.
<br/>
<br/>
*Not surprisingly, today, two days after, the monitor was still not working!*
<br/>
<br/>
Now, luckily this crap system is not driving the train but simply providing information about the route and the timetable, because it is quite embarassing to still have this annoying problems on a modern system.
