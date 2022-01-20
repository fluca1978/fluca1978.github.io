---
layout: post
title:  "Useless Uses of Cat (teaching of)"
author: Luca Ferrari
tags:
- linux
- university
permalink: /:year/:month/:day/:title.html
---
A tale back from my university years...

# Useless Uses of Cat (teaching of)

When I was a green, young, nervous university student, I had a professor that was in charge of the *Operating Systems* course.
The course was based, as you could guess, on Unix (and Linux).
<br/>
One of the activity to learn within the course was the interaction with the Unix shell, with particular regard to the Bourne Shell.
<br/>
The professor was addicted to **pipelining** commands together as a *superior way to get things done by means of small tools*. After all, this is somehow the Unix philosophy.
<br/>
There is, however, a drawback: **piping commands together requires the shell to open a new process and interconnect both of them**, and moreover this requires extra activity on the scheduler machinery. While creating a new process is fast, especially nowdays, this is not a good way of doing things in Unix.
<br/>
In fact, one thing that was not tought to us, was that many commands did accept several forms of input beside `stdin`, and therefore there was no useful usage of concatening them together in many situations.

<br/>
In the following, I will try to demonstrate with a couple of simple examples, what I mean with the nonsense concatenation of applications.

## List only directories

The solution, according to my professor, was to use `ls` and `grep`: `ls -l | grep ^d`.
<br/>
*Two processes involved, and also a regular expression*! Uhm, this does not look simple to me.
<br/>
So, what is an alternative solution? **RTFM**!
<br/>
`ls` contains a flag, `-d`, that can be used combined to `l` as: `ls -ld */`.

## Count the occurrencies of lines containing a specific string

She aggressively proposed to do: `cat filename.txt | grep university | wc -l`.
<br/>
Three processes!
<br/>
`grep` has a particular flag, `-c`, to count occurrencies lines: `grep -c university filename.txt`.


## Useless Uses of `cat`

There are other examples, but the above demonstrates how wrong it was the teaching of the Unix and shell pipeline.
However, there are places where it does make sense to have a pipe between commands: when you don't want you (right) command to know on which it is working.
<br/>
Allow me to explain with an example:

<br/>
<br/>

``` shell
% grep university *
file.txt:university
file.txt:university
filename.txt:university
filename.txt:university
filename.txt:university


% cat * | grep university
cat: a1: Is a directory
cat: b2: Is a directory
university
university
university
university
university

```
<br/>
<br/>

Did you spot the difference in the two commands?
<br/>
The first one, not using the pipe, **shows the filename where the occurencies is found**, while the second command **does not show which file contains the occurrency**.
<br/>
Why? Because the first execution makes the command aware of the input file to open, and thus can produce a more detailed information as output. The second command, on the other hand, collects the whole content into a single **un-named `stdibn`** that is used as the content for the second rigthmost command, that in turn does not know about the source of the content.

# Conclusions

The term *useless uses of `cat`* is often related to the *pipeling commands on steroids*.
<br/>
This way of combining different commands is surely a powerful feature of every shell, and is a way to build complex tools. However, in my opinion, this does make sense only when there is the need to *hide* the source of the content to a command.
<br/>
Often, the solution you are looking for is a flag away on a single command. And a single command means less effort, both for you and for the machine!
