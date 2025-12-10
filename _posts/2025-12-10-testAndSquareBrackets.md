---
layout: post
title:  "Test your square brackets!"
author: Luca Ferrari
tags:
- FreeBSD
- Linux
- Bash
permalink: /:year/:month/:day/:title.html
---
A shell story about how to test things when doing scripting.

# Test your square brackets!

I do shell scripting everyday, and I'm so used to the usage of square brackets when performing some test, that I forgot the whole story.
I assume you know that something like the following performs what it seems to do:

<br/>
<br/>
```shell
if [ -f "$file" ]; then
   echo "The file exists, avoid overwriting it!"
   exit 2
fi
```
<br/>
<br/>

The important part here is the `[ ... ]` test line.

Let's turn on the DeLorean circuits, and go back in time!


## `test`, as it was and it always will be

When I was a young, green, university student, I was forced to use `test(1)` as the **only true way** to do testing in shell scripting.
Therefore, the above snippet of code, was written as follows in my homework:

<br/>
<br/>
```shell
if test -f "$file"
then
  ...
fi
```
<br/>
<br/>

Yeah, I was also forced to *not use semicolons* as they were evil (according to my professor, any comment unneeded!).

So, what is that `test` thing after all?
I started exploring, and found out that `test` was a regular command lying somewhere in `/usr/bin` or `/bin` (or both, via a link).

It's simple enough to test `test` (no pun intended!) on a shell command line (i.e., outside a script):

<br/>
<br/>
```shell
% test -d /home/luca

% echo $?
0

```
<br/>
<br/>

So far, so good.


## `[` and the modern era

Then, while reading a Linux specific book, I found out that it was possible to use the `[ ... ]` syntax to perform something similar to what I did in C, like `if ( ... )`, where the parenthesis were substituted by brackets.
Digging a little more, I found out that the `[` was itself a command, pretty much as `test`.
Then, doing a `ls -l` I discovered that `test` and `[` were much more than similar programs, they were the same program:

<br/>
<br/>
```shell
% ls -l '/bin/test' '/bin/['
-r-xr-xr-x  2 root wheel 12184 Jun  6  2025 /bin/[
-r-xr-xr-x  2 root wheel 12184 Jun  6  2025 /bin/test

% sha1sum '/bin/test' '/bin/['
623e9bbe1784ebf1c8fb5d404978e7ffb7b1ef7b  /bin/test
623e9bbe1784ebf1c8fb5d404978e7ffb7b1ef7b  /bin/[

```
<br/>
<br/>

Therefore, when placing a `[` into my scripts, I was silently calling `test`.


## Understanding the closing bracket

So, after having discovered that `[` was just an handy shortcut for `test`, where did the closing bracket come from?
In the beginning I thought it was a shell syntax enforcement, but I was wrong.

The turth is that `test`, ehm, sorry, `[` knows how it has been invoked and in the case it has invoked as `[` it checks for the very last argument to be a closing bracket, otherwise an error is thrown.

This is [clearly reported on the FreeBSD implementation](https://github.com/freebsd/freebsd-src/blob/main/bin/test/test.c#L199){:target="_blank"} of `test`:

<br/>
<br/>
```c
if (strcmp(p, "[") == 0) {
		if (strcmp(argv[--argc], "]") != 0)
			error("missing ']'");
		argv[argc] = NULL;
	}
```
<br/>
<br/>

If the command has been invoked as `[` and the last argument `argv[--argc]` is not `]`, buhm, error!


## The devil is in the small details

Yesterday evening [I was at a Bash course at the LUG of my city](https://conoscerelinux.org/courses/corso-bash/){:target="_blank"}, with a friend of mine.
Suddenly I remembered the whole story above, and how I was excited to prove that `[` and `test` were the same thing.
However, I found out that, on many (almost all) Linux machines these days the two commands are not anymore the same:

<br/>
<br/>
```shell
 ls -l '/usr/bin/test' '/usr/bin/['
-rwxr-xr-x 1 root root 55744 apr  5  2024 '/usr/bin/['
-rwxr-xr-x 1 root root 47552 apr  5  2024  /usr/bin/test

% sha1sum '/usr/bin/test' '/usr/bin/['
aba4f68d151e50aff3b4d2d6213ae6b1fd114255  /usr/bin/test
3d0fa425ddd0aca6fc4b13dcc90db07b8b53c5b0  /usr/bin/[

```
<br/>
<br/>

It turns out that *GNU coreutils* prefers to build the two identical programs with different compilation flags, even if the [applications share the same source file](https://github.com/coreutils/coreutils/blob/master/src/test.c#L828){:target="_blank"} and the test within it is really similar to the one that FreeBSD does.



## University sad story about this

This whole thing about `[` and `test` is really clear in my mind, even if I tend to keep it into the back of my mind memory, because it caused me a very poor evaluation of my university exam.
In fact, back in those days, I thought that having found by myself that `test` and `[` are the same application (and yes, they were linked together even in Linux, back then), I use `[ ... ]` in the exam script.

My professor was very unhappy, and encounted a single error **every time I used `[` instead of `test`**, resulting in my exam getting a final vote of `22` out of `30`, where *six* errors where due to this substitution. I argued that, having the university laboratory a Linux based set of machines, I was testing the scripts on the only machines I had access to, but no matter, since my script was not able to run on the professor Sun OS `I-don't-remember-which-version` Bourne Shell, my exam did not get any chance to get a better evaluation.


# Conclusions

Simply enough to remember: `test` and `[` **are the same thing even if they do not appear to be** (i.e., are different executables), and the syntax is chacked directly by the application itself, that decides if the closing bracket is required or not.
