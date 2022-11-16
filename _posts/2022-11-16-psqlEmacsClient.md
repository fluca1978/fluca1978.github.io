---
layout: post
title:  "Emacs(client) as editor in psql"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
At last I found a way to use Emacs (as a client) within psql!

# Emacs(client) as editor in psql

**psql** is an amazing interactive SQL swiss-army knife terminal thingy that can really tunr your day! Quite frankly, in my professional training activity, I always tell participants to learn to use `psql` for several reasons, and I also ask them if any of their interactive SQL-terminals has the same set of features, without having back an answer!


One nice thing that `psql` provides, is the capability to edit a complex query directly within your editor of choice.

My *editor of choice* is **Emacs**!

Starting Emacs every time I have to edit a query buffer (via `\e` command) is awkward: even on recent hardware, Emacs startup is slow.
Thankfully, there is a trick: *Emacs embeds a client-server approach*. And it is a **real** client-server approach.

The idea is to have Emacs started as a *daemon*, and every time you need a new editor frame (ehm, every time you need to edit something), you can invoke the special command `emacsclient` asking to attach to the daemon running. As a result, while the daemon startup is as slow as a normal `emacs` instance is, the `emacsclient` editing session is shiningly fast!

So far so good! Ehm, no, well, not for me. I had a lot of troubles in trying to configure `psql` to use `emacsclient` as an editor. And, *shame on me*, I had troubles because I was using the wrong set of flags to launcha `emacsclient`! To make things worst: I'm using *ZSH* as my default shell, and for some strange reason I need to investigate on, the shell does not work really well with *commands and flags within the same environment variable*.

## TL;DR - How to do that?

There are two ways to change the default editor that `psql` is going to use to edit a query buffer:
- set the well known `EDITOR` environment variable;
- set the `psql` specific `PSQL_EDITOR` environment variable.

Since I'm a whole-Emacs kind of guy, I decided for the first, so that whenever I need to use `$EDITOR`, I will be dropped into my comfortable Emacs environment.

Therefore, in order to achieve this, after some research on the flags, simply put the following in your `.zshrc` configuration file:

<br/>
<br/>
```shell
export EDITOR="emacsclient -t"
```
<br/>
<br/>

or do the same with `PSQL_EDITOR`.

Then, when you are in the `psql` application, hit `\e` and look at Emacs quickly appear. The very first time, when you will come back to your `psql` session, you will notice a few lines of warnings coming out from Emacs itself:

<br/>
<br/>
```shell
testdb=> \e
emacsclient: can't find socket; have you started the server?
emacsclient: To start the server in Emacs, type "M-x server-start".
Starting Emacs daemon.
Emacs daemon should have started, trying to connect again
testdb=>

```
<br/>
<br/>

That's because, at the very first time, `emacsclient` does not find any `emacs` instance running as a daemon, so it starts one by itself, waits a moment, and connect back to the server. This is clearly explained in the messages, that are therefore just warnings.


## What about ZSH (and Bash)?

One of the reason it took me so long to understand how to configure `emacsclient`, was that for some reasons I don't know (yet), ZSH behaves nastly when you try to launch a command within an environment variable, assuming such variable contains spaces. Why? Because in order to set the variable with spaces, you have to quote your command line, and at that point ZSH assumes the whole string is a single command:


<br/>
<br/>
```shell
% echo $EDITOR
emacsclient -t

% $EDITOR
zsh: command not found: emacsclient -t
```
<br/>
<br/>


The same does not happen in Bash, that behaves as expected:


<br/>
<br/>
```shell
% bash
$ echo $EDITOR
emacsclient -t
$ $EDITOR
...
```
<br/>
<br/>

So, I've also found a place where Bash seems to behave more intuitively than ZSH is (and *no, I'm not going to switch back to Bash for this reason!*).


## Colors and themes!

I tend to use a dark color scheme in all my activities, included terminals I use to SSH-in machines. In these circumstances, Emacs has a very poor default color choice.
Assuming you don't have a specific Emacs startup configuration (and you should, believe me!), you can add a line like the following to one of your startup files (e.g., `~/.emacs` or `~/.emacs.d/init.el`):

<br/>
<br/>
```lisp
(load-theme 'tango-dark)
```
<br/>
<br/>

This will make your Emacs experience on dark terminals a lot better!

Please consider that changes will not be reflect into any running daemon, so you have to restart Emacs (or re-evaluate) the startup files once you have changed!


# Conclusions

Emacs, thanks to its client-server component, can really improve your already awesome experience within `psql`!
For instance, you can use external tools to reformat your [queries while you are writing them in the editor](https://fluca1978.github.io/2022/04/13/EmacsPgFormatter.html){:target="_blank"}.

For more documentation on how to customize `emacsclient` [see the official documentation](https://www.emacswiki.org/emacs/EmacsClient){:target="_blank"}.
