---
layout: post
title:  "Eclipse and go-to-line"
author: Luca Ferrari
tags:
- eclipse
- java
permalink: /:year/:month/:day/:title.html
---
I discovered a new thing about the Eclipse status bar.

# Eclipse and go-to-line

Today, while I was developing as often within the Eclipse IDE, I noted a strange thing at the bootom, in the *status bar*:

<br/><br/>
<center>
<img src="/images/posts/eclipse/status_bar.png" />
</center>
<br/><br/>

The row and column indicator has something strange to me: instead of showing two values (one for the current row and one for the current column), it has three values!
<br/>
In the previous example, it has `265 : 1 : 6847`.
<br/>
The meaning is as follows:
- the first number is the *row number*;
- the second number is the *column number*;
- the third number is the number of bytes of the file.

<br/>
Therefore, `265` is the current row, `1` is the current column and `6847` is the number of characters of the file.
<br/>
I think the information could be displayed a little better, for example making the file size (in characters) less intrusive, since it is not very interesting, at least to me, while developing or writing code.


# Go-To-Line

I discovered also another thing: **if you click on that portion of the status bar the `go-to-line` window pops-up**.


<br/><br/>
<center>
<img src="/images/posts/eclipse/go_to_line.png" />
</center>
<br/><br/>


