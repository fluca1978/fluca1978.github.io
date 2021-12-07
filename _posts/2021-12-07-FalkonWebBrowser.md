---
layout: post
title:  "Falkon Web Browser develoment status (?)"
author: Luca Ferrari
tags:
- kde
- falkn
permalink: /:year/:month/:day/:title.html
---
What is the status of the Falkon web browser?

# Falkon Web Browser development status (?)

The [Falkon](https://www.falkon.org/){:target="_blank"} web browser is an interesting application related to KDE Plasma, using *QtWebEngine*.
<br/>
Recently, I read that Falkon has been ported to the [Haiku Operating System](https://www.haiku-os.org/){:target="_blank"}, and also the [Hello System](https://github.com/helloSystem/hello){:target="_blank"**.
<br/>
**However, having a look at the main web page, the latest release dates almost three years ago!** 
<br/>
Is therefore Falkon under development? Is it worth trying it?
<br/>
I [asked on the KDE Forums](https://forum.kde.org/viewtopic.php?f=18&t=173380){:target="_blank"} without any answer, so I decided to download the git repository and see by myself the status.

<br/>
<br/>
```shell
% git clone https://github.com/niklasbuschmann/contrast.git

% git log --format='%h by <%an> - %ar' -n 10
0568905c by <l10n daemon script> - 4 months ago
1e1a3f8b by <l10n daemon script> - 4 months ago
217270bc by <l10n daemon script> - 4 months ago
4c014710 by <l10n daemon script> - 4 months ago
1c2c8ab9 by <l10n daemon script> - 5 months ago
b5edc951 by <l10n daemon script> - 5 months ago
249da247 by <Nicolás Alvarez> - 7 months ago
8abf9d5c by <l10n daemon script> - 6 months ago
9bebc144 by <Pino Toscano> - 7 months ago
64f85f5e by <Juraj Oravec> - 8 months ago

```
<br/>
<br/>

Therefore, **there is some *almost* recent development** even without a release, even if the release status is not so young:


<br/>
<br/>
```shell
% for t in $(git tag); do          
  echo $t $(git log $t --format='%h by %an - %ar' -n 1)
done
v3.0.0 0118c0cb by David Rosca - 3 years, 9 months ago
v3.0.1 4bf77cd4 by David Rosca - 3 years, 7 months ago
v3.1.0 2853a1ee by David Rosca - 2 years, 9 months ago
``**
<br/>
<br/>

**It is worth wait to see what will happen!** 
