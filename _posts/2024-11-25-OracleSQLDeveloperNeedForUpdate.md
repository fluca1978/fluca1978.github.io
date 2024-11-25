---
layout: post
title:  "Oracle SQLDeveloper not starting: need for an update?"
author: Luca Ferrari
tags:
- oracle
permalink: /:year/:month/:day/:title.html
---
Today my instance of SQLDeveloper refused to start...

# Oracle SQLDeveloper not starting: need for an update?

The title says it all: today my instance of SQLDeveloper refused to start.
It initially complained about the need to update some modules, that I agreed to, but then nothing happens at all.

Therefore, I decided to start it from the terminal, to get some clue, and in fact, the stacktrace revealed the horror:

<br/>
<br/>
```shell
% /opt/sqldeveloper/sqldeveloper.sh

Oracle SQL Developer
Copyright (c) 2005, 2024, Oracle and/or its affiliates. All rights reserved.

OpenJDK 64-Bit Server VM warning: Options -Xverify:none and -noverify were deprecated in JDK 13 and will likely be removed in a future release.
WARNING: A terminally deprecated method in java.lang.System has been called
WARNING: System::setSecurityManager has been called by org.netbeans.TopSecurityManager (file:/opt/sqldeveloper/netbeans/platform/lib/boot.jar)
WARNING: Please consider reporting this to the maintainers of org.netbeans.TopSecurityManager
WARNING: System::setSecurityManager will be removed in a future release
WARNING: A terminally deprecated method in java.lang.System has been called
WARNING: System::setSecurityManager has been called by oracle.ide.IdeCore (file:/opt/sqldeveloper/ide/extensions/oracle.ide.jar)
WARNING: Please consider reporting this to the maintainers of oracle.ide.IdeCore
WARNING: System::setSecurityManager will be removed in a future release
Gtk-Message: 10:08:12.115: Failed to load module "appmenu-gtk-module"

(java:17241): Gtk-WARNING **: 10:08:12.122: Unable to locate theme engine in module_path: "adwaita",

```
<br/>
<br/>

So, there was a call to something broken in Java-land.

Without any need to dig further, I decided to upgrade my SQLDevelop to version `24.3.0` and now it works as usual over my Java 17.
