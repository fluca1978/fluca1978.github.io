---
layout: post
title:  "Mojolicious: how I messed up with the flash!"
author: Luca Ferrari
tags:
- perl
- mojolicious
permalink: /:year/:month/:day/:title.html
---
Finally I got the explanation of what I was doing wrong with `flash`.

# Mojolicious: how I messed up with the flash!

I *used to abuse* [Mojolicious](https://mojolicious.org/){:target="_blank"} structures, like the *stash*, managing them as Perl hashes.

And it worked!

*Unless when it does not work!*

The `flash` was a particular case that did not worked for me, until I finally got a [very good and detailed answer to a question](https://github.com/mojolicious/mojo/discussions/2240#discussioncomment-12789448){:target="_blank"}.

TL;DR: I was using the `flash` as follows:

<br/>
<br/>
```perl
$self->flash->{ message } = 'Hello from flash';
```
<br/>
<br/>

and that did not worked. As explained in the above Github issue, and after having a look at the very concise code in `Mojolicious::Sessions`, with particular regard to the `store` and `load` methods, the `flash` is substituted at every request with a *new one*, hence the above is not working.
For example, in `load`:

<br/>
<br/>
```perl
sub load {
...
$session->{flash}        = delete $session->{new_flash} if $session->{new_flash};
}

```
<br/>
<br/>


So, what is the solution to the problem? **Just use `flash` method to set the `flash` status**, so that internally the `new_flash` will be set and will override the *current* `flash`:

<br/>
<br/>
```perl
$self->flash( message => 'Hello from flash' );

```
<br/>
<br/>
