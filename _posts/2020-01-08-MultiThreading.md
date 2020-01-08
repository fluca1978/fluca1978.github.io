---
layout: post
title:  "General Rules in Multithreading"
author: Luca Ferrari
tags:
- Java
- Qt
- multithreading
- aglets
- university
permalink: /:year/:month/:day/:title.html
---
I read a blog post about multithreading and found it useful.

# General Rules in Multithreading

Today I've read this [very good blog post about using multithreading in Qt](https://www.kdab.com/the-eight-rules-of-multithreaded-qt/){:target=_blank}, and even if today I don't manage threads by myself, that reminded me my first attempt in understanding and explaining the multithread.
<br/>
<br/>
My first attempt in doing multithread was while studying (by my own) Java. It was Java 1.1, and the language included a very simple interface to multithreading, so it was quite simple to launch an applet that spawn a few threads to do some flash GUI updates.
<br/>
Later I discovered other Java system were, of course, using multithreading. And while I had to use, in my university, other multithreaded C libraries, my main focus on multithreading was in Java.
<br/>
<br/>
Back in those days, multithreading seemed to me really easy: you create a new thread, synchronized other a shared *mutex*, and finalize the thread once it has done its job.
<br/>
<br/>
So far, so good!
<br/>
<br/>
But then came the GUI development.
<br/>
Of course, a GUI application as a few threads running for you and handling all the small details. But when you *are addicted* to *spawn-synchronize multiple threads* you tend to do the same even in GUI applications, and you application freezes!
<br/>
It is obvious, once you understand it: a GUI application is a thread-based application, but it does synchronization by *message passing*. Some languages calls it *events*, other *event loop*, and so on. The idea is simple: your main thread is processing stuff out of an event queue.
<br/>
The rule of thumb of the above is that **you do not block the event thread** or all your application will freeze.
<br/>
With "blocking" I mean either put to sleep or let it do a very intensive task (e.g., updating a lot of rows on a database connection, download a file, and so on).
<br/>
<br/>
I learnt the above on my own experience, luckily while I was still a little student at university, but I have to confess that I made the same mistake on my day-to-day job too! Therefore, I was really glad to see the blog post placed the same advice as the first items on the list:
1) *Never call QThread::sleep()*
2) *Never do GUI operations off the main thread*
3) *Don’t block the main thread*
<br/>
<br/>
Of course, the above advices does not apply only on GUI applications, but anywhere you have an event-loop based mechanism. In fact, [I developed the `suspend()` method in Aglets](https://sourceforge.net/p/aglets/git/ci/master/tree/){:target=_blank} to emulate the `Thread.sleep()` method, making an agent something more similar to a thread.
