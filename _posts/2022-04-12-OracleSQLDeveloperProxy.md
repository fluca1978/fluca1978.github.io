---
layout: post
title:  "Oracle SQL Developer and the Proxy Users"
author: Luca Ferrari
tags:
- oracle
- java
permalink: /:year/:month/:day/:title.html
---
Oracle SQL Developer and strange behaviour about proxy users...

# Oracle SQL Developer and the Proxy Users

I upgraded my Oracle SQL Developer to version `21.4.3`, and when started it asked me to *import* previous connections from the previous version.
<br/>
I clearly answered *Yes* (of course!).
<br/>
No connection was working thought!
<br/>
There were two problems, that took me a while to understand:
- proxy users have a dedicated configuration now;
- for some reason the SQL Developer defaults to use *SID* instead of *Service Name* even when the latter was defined in the previous configuration definition.

## The problem with the proxy users

A proxy user is, in short, a way to let Oracle connect you in a database that is different from your own username. Usually, proxy users use a syntax liek `username[schema]`, that is well accepted by the instant client.
<br/>
With newer versions of Oracle SQL Developer the proxy user configuration has been split into a different panel of the configuration setting. Therefore you have to:
- define your own username in the authentication panel, within the *Username* textbox;
- insert you own password, if you want it to be stored;
- swith to the *Proxy User* panel and check the checkbox `Use  DB Proxy Authentication`, then insert the schema name into the *Proxy Client* text field.

<br/>
<br/>
<center>
<img src="/images/posts/oracle/sqldeveloper_proxy_user_1.png" />
<br/>
<img src="/images/posts/oracle/sqldeveloper_proxy_user_2.png" />
</center>
<br/>
<br/>

otherwise **using the same syntax allowed by the InstantClient, you will not be able to connect!**.


## The problem with the SID

Ensure to use the service name in the connection information, because even if I think the SID is less and less used every day, the connection dialog defaults to using the SID.
<br/>
Even when importing existing connections that already use the *Service Name*!

## The problem with Java

There is a little strange fact: Oracle SQL Developers claims to work with a JVM no greater than `11`!
<br/>
Wait a minute, aren't you the same Java company?
<br/>
So far, for me, the application is working with Java `17`, but it clearly states that there could be problems.


# Conclusions

Oracle SQL Developer is the less flexible database client I know, so far.
