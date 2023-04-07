---
layout: post
title:  "How much does it take to compile PostgreSQL (on my machines)?"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A few considerations on how fast (or slow) it can be to compile PostgreSQL

# How much does it take to compile PostgreSQL (on my machines)?

A couple of months ago I bought a new laptop, an *Acer Aspire 5 A515-45-R9EC*, with an *AMD Ryzen 5 500U* CPU.
**It is the first Ryzen processor I never had**, so I was curios to see how it performs, and what a better test approach for me than compiling PostgreSQL from scratch?
<br/>
I fired up [pgenv](https://github.com/theory/pgenv){:target="_blank"} and did a simple `pgenv build 15.2` to compile the whole PostgreSQL 15.2 distribution. I was quite happy with my results, and I [posted the following](https://www.linkedin.com/posts/fluca1978_emacs-postgresql-activity-7042159450253602816-cYAC?utm_source=share&utm_medium=member_desktop){:target="_blank"}

<br/>
<br/>
```
My new "mule" laptop with AMD Ryzen 5 5500U
compiles #emacs 28.2 in 86 secs and #PostgreSQL 15.2 in 226 secs. There's no time for a coffee anymore!
```
<br/>
<br/>

Then a friend of mine pointed out that they were not great times, sob!
<br/>
This triggered some curiosity, so I decided to compare my three hardware machines to see how they perform. I report a single time example, but all times are pretty much stable and could change only by means of a couple of seconds (in other words, *this is not a real benchmark!*):

<br/><br/>

| CPU                | cores | thread | compilation type | seconds |
|:------------------:|:-----:|:------:|:----------------:|:-------:|
| Intel i5-5257U     | 2     | 4      | make -j          | 166     |
| Intel i5-10500T    | 6     | 12     | make -j          | 57      |
| AMD Ryzen 5 5500 U | 6     | 12     | make -j          | 61      |


<br/><br/>

The usage of `make -j` is to use all the available parallelism possible on the machine, and in the meantime all the computers were not doing anything else.
The slowest is, as obvious, the machine with two cores. It is interesting to note that all the machine are *low energy consumption*, therefore they are not performing as well as a desktop or server environment.

<br/>
I then decided to comile another *beast*: my favourite editor *Emacs 28.2*!

<br/><br/>

| CPU                | cores | thread | compilation type | seconds |
|:------------------:|:-----:|:------:|:----------------:|:-------:|
| Intel i5-5257U     | 2     | 4      | make -j          | 76      |
| Intel i5-10500T    | 6     | 12     | make -j          | 31      |
| AMD Ryzen 5 5500 U | 6     | 12     | make -j          | 33      |


<br/><br/>

Again, times are not so bad after all. But the above results are with the maximum parallelism, what happens in normal conditions?

<br/><br/>


| CPU                | cores | thread | compilation type | seconds to compile PostgreSQL 15.2 | seconds to compile Emacs 28.2 |
|:------------------:|:-----:|:------:|:----------------:|:----------------------------------:|-------------------------------|
| Intel i5-10500T    | 6     | 12     | make             | 211                                | 106                           |
| AMD Ryzen 5 5500 U | 6     | 12     | make             | 193                                | 92                            |



<br/><br/>

As a final note, if I virtualize the `Intel i5-10500T` as a single CPU with as a single core, the time to compile PostgreSQL grows to 250 seconds. Clearly, such time is comparable with the `make` sequential on the same CPU.
