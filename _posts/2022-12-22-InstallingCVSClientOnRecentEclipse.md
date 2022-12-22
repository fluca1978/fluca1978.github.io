---
layout: post
title:  "Installing CVS Client on a Modern Eclipse IDE"
author: Luca Ferrari
tags:
- eclipse
- java
- cvs
permalink: /:year/:month/:day/:title.html
---
Eclipse dropped almost a year ago the support for CVS.


# Installing CVS Client on a Modern Eclipse IDE

With the `2021-12` release, Eclipse IDE dropped the integrated support to CVS: that was the last known release to have an installable client for the *Concurrent Versioning System*.
<br/>
If you have a *recent* version of Eclipse, e.g., `2022-09`, you will not be able to find a CVS client on the `Install new software` available sites.
The solution is to manually add the latest repository known that includes the CVS client, that is **`https://download.eclipse.org/eclipse/updates/4.22`** and the search it into such a repository:

<br/>
<center>
<img src="/images/posts/eclipse/cvs_repository.png" />
</center>
<br/>
<center>
<img src="/images/posts/eclipse/cvs_client.png" />
</center>
<br/>

I'm not sure this solution will work for future versions of Eclipse, and quite frankly, while I understand that CVS is somehow an old versioning system, it is still widely used, and removing an integrated client could make people unhappy.
