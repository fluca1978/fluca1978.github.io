---
layout: post
title:  "PHP, SimpleXLS and why I Love Open Source"
author: Luca Ferrari
tags:
- php
- opensource
permalink: /:year/:month/:day/:title.html
---
I found a bug in a PHP module I used to parse Excel-like files.

# PHP, SimpleXLS and why I Love Open Source

I [found a curious bug](https://github.com/shuchkin/simplexls/issues/3){:target="_blank"} in the [SimpleXLS](https://github.com/shuchkin/simplexls){:target="_blank"} module that I used to parse Excel files in my PHP applications.
<br/>
The bug was quite impressing: a negative number different than `-1` caused it to be parsed as a very high positive number. I tried to dig the code in order to both fix the problem and find information useful to compile the bug report, and it seemed to me there was a problem of *two's complemention*.
<br/>
As far as I can understand [from the applied patch](https://github.com/shuchkin/simplexls/blob/master/src/SimpleXLS.php#L421){:target="_blank"}, it was effectively a *two's complemention* problem on the value.
<br/>
<br/>
I have to confess, being this module appropriate for *Microsoft Excel 97*, a file format **22 years old**, I was not really hopeful in a solution at all. I don't want to be harsh, but quite frankly there are out there a lot of other excellent modules that do the very same stuff, and while I like [SimpleXLS](https://github.com/shuchkin/simplexls){:target="_blank"} for being sweet and to the point, I started thinking to use another module.
<br/>
<br/>
However, **in last then 23 hours I got a feedback** and a [new version of the module was released](https://github.com/shuchkin/simplexls/releases/tag/0.9.5){:target="_blank"}**. 

I quickly tried it, and it worked!
<br/>
<br/>
**I have to really thank the module author** and I cannot emphasize enough how important the Open Source can be! If you don't believe me, go and try yourself to have the same support from a private company and report it back!

<br/>
<br/>
<br/>
<br/>
In any case, to be really honest, the bug made me switch from SimpleXLS and SimpleXLSX to `PHPOffice`. The bug has been just a spark to the change, the real reason was that with PHPOffice I am now able to use the very same module for different *Excel-like* formats instead of using two different modules. Anyway, I will use SimpleXLS in other projects when a single format is needed.


