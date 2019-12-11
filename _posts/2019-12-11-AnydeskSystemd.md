---
layout: post
title:  "AnyDesk always showing up in my desktop"
author: Luca Ferrari
tags:
- kde
- anydesk
- systemd


permalink: /:year/:month/:day/:title.html
---
I hate applications prompting on the desktop session starting phase.

# AnyDesk always showing up in my desktop

A few days ago I installed [AnyDesk](https://anydesk.com){:target="_blank_"}, an application to remotely control a desktop session. I like it, it is lightweight and fast, and as opposite to other similar products does not brutally close your connection to force you to *stay non-profit (or buy a commercial license)* (yes *TeamViewer* I'm talking about you!).
<br/>
<br/>
So far so good, but at the very next desktop login the AnyDesk icon was there in the system tray.

<br/>
<br/>
<center>
<img src="/images/posts/anydesk/anydesk1.png" />
</center>
<br/>
<br/>
I checked the *Autostart* configuration of my KDE desktop (ops, I should say *Plasma desktop*), but there was nothing there.
<br/>
Uhm...so I guessed there was a systemd entry.
<br/>
And I was right: **AnyDesk installs a `systemd` service file**.

<br/>
<br/>
<center>
<img src="/images/posts/anydesk/anydesk2.png" />
</center>
<br/>
<br/>

Therefore, the solution to avoid anydesk to start on every login is quite simple:

```shell
% sudo systemctl disable anydesk
Removed /etc/systemd/system/multi-user.target.wants/anydesk.service.
```

And my desktop is happy again!

## Considerations

I was very well impressed by the choice of using `systemd` for AnyDesk. Usually, other applications place a startup script into the *autostart* desktop mechanism, and while `systemd` is aggressively Linux specific, and I'm not a fan of `systemd`, I believe that to the purpose of AnyDesk the choice is right.
<br/>
After all, if you are using to share your own desktop, then having it installed and started by means of a service is the bettermost approach to do.
<br/>
What I'm not comfortable at, is the choice made by the application to automatically start at boot time.
