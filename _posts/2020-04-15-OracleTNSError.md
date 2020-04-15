---
layout: post
title:  "ORA-12514 and the mispelled TNS"
author: Luca Ferrari
tags:
- oracle
- java
permalink: /:year/:month/:day/:title.html
---
The importance of right error messages!


# ORA-12514 and the mispelled TNS

This happened to me while trying to start a Java application:

```
Caused by: java.sql.SQLException: Listener refused the connection with the following error:
ORA-12514, TNS:listener does not currently know of service requested in connect descriptor
```

<br/>
<br/>
What the hell was going on?
<br/><br/>
Well, I simply *mispelled the name of the service in the connection file*, that means I forgot a service name letter in the JDBC URL for the connection.
<br/><br/>
That clearly means **it is my fault**, but wait a minute, isn't this a *poor error message* without any particular specification about the error I made? Why neither the Oracle driver or the JDBC connector reports back the mispelled name, since they are able to tell me that such name does not exist?
