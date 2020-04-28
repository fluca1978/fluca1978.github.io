---
layout: post
title:  "Oracle SQL Developer and the Assistive Technology Exception"
author: Luca Ferrari
tags:
- oracle
- java
- linux
- kubuntu
permalink: /:year/:month/:day/:title.html
---
A strange error with Java on Ubuntu.


# Oracle SQL Developer and the Assistive Technology Exception

I migrated my Oracle SQL Developer installation after my hard drive did crash.
<br/>
Of course, this is something you should not do, and should install everything from scratch, but I'm lazy!
<br/>
However, launching the java application I got an exception:


```shell
% /opt/sqldeveloper/sqldeveloper.sh                        

 Oracle SQL Developer
 Copyright (c) 1997, 2015, Oracle and/or its affiliates. All rights reserved.

Exception in thread "main" java.awt.AWTError: Assistive Technology not found: org.GNOME.Accessibility.AtkWrapper
        at java.awt.Toolkit.loadAssistiveTechnologies(Toolkit.java:807)
        at java.awt.Toolkit.getDefaultToolkit(Toolkit.java:886)
        at java.awt.Toolkit.getEventQueue(Toolkit.java:1736)
        at java.awt.EventQueue.isDispatchThread(EventQueue.java:1071)
        at javax.swing.SwingUtilities.isEventDispatchThread(SwingUtilities.java:1366)
        at oracle.ide.osgi.boot.SplashScreenImpl.SynchronizeWithEdt(SplashScreenImpl.java:531)
        at oracle.ide.osgi.boot.api.SplashScreen.createInstance(SplashScreen.java:66)
        at oracle.ide.osgi.boot.OracleIdeLauncher.showSplashScreen(OracleIdeLauncher.java:821)
        at oracle.ide.osgi.boot.OracleIdeLauncher.main(OracleIdeLauncher.java:113)
Error: SQL Developer can't recognize the JDK version
```

The JDK was running just fine, I've Eclipse running on top of it.
<br/>
So to fix the problem I had to edit the `/etc/java-8-openjdk/accessibility.properties` commenting out the line about GNOME assistive implementation (I'm running Plasma):

```shell
% sudo cat /etc/java-8-openjdk/accessibility.properties

#
# The following line specifies the assistive technology classes 
# that should be loaded into the Java VM when the AWT is initailized.
# Specify multiple classes by separating them with commas.
# Note: the line below cannot end the file (there must be at
# a minimum a blank line following it).
#
#assistive_technologies=org.GNOME.Accessibility.AtkWrapper

```

<br/>
While I probably will need assistive technologies, let not assume I'm using GNOME!
