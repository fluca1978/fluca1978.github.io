---
layout: post
title:  "Xiaomi and MIUI: where are my pictures?" 
author: Luca Ferrari
tags:
- linux
- android
permalink: /:year/:month/:day/:title.html
---
When Linux hides things to Linux...

# Xiaomi and MIUI: where are my pictures?

I own a *Xiaomi Redmi 9* phone. It is a decent phone, it does its job, and I'm quite satisfied.
<br/>
Sometimes I do backup my photos from the phone to my hard drives, yes, I don't do put them in the cloud!
<br/>
But today I was not really happy: once I plugged the USB cable, I was unable to find out all my albums and photos. Of course, I knew they were there, but where?
<br/>
It turned out, after a deep introspection, that for some reason the *MIUI Android Operating System* stores the pictures into a *strange* path within a folder called `MIUI`, in particular into `MIUI/gallery/cloud/owner`. Please note that there is a `cloud` subfolder even if I don't have the cloud sharing enabled, I suspect the application has been defined with an hardcoded path because the cloud sharing is probably the most often used setup.

<center>
<br/>
<br/>
<img src="/images/posts/kde/xiamoi_redmi9_photos_dolphin.png" />
<br/>
</center>

This behaviour reminds me some kind of *vendor lock-in*, so that one day you will find out it is easier to interact with your phone by some sort of Xiaomi application, but until then I will still use my own Linux tools to access my data locally!
