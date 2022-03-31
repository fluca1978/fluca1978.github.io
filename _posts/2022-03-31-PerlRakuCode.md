---
layout: post
title:  "Anonymous block of code in Perl and Raku"
author: Luca Ferrari
tags:
- perl
- raku
permalink: /:year/:month/:day/:title.html
---
Little differences between the usage of anonymous blocks.

# Anonymous block of code in Perl and Raku
In both Perl and Raku you can specify anonymous blocks containing code, sometimes also a *closure*, store them into variables and use later on.
<br/>
As an example, I use often when dealing with Pl/Perl code within PostgreSQL in order to avoid overhead of calling other (SQL) functions.
<br/>
Raku being, according to me, *Perl on sterooids*, has some tricks to use in a more readable way such blocks.

## Perl Code Blocks

A simple example of available invocations:

<br/>
<br/>

``` perl
my $action = sub { say "Hello $_[0]"; };

$action->( 'fluca1978' );
&{ $action }( 'fluca1978' );

```
<br/>
<br/>

You can choose your preferred way to call the *action* code using only the arrow (dereferencing) or the well known `&` sigil (that indicates subroutines).

## Raku Code Blocks
 In Raku the derefencing operator is the `.`, so the first invocation is really similar to the Perl one. However, the `&` sigil allows you to manage the variable exactly as a function:

 <br/>
 <br/>
``` raku
my $action = { "Hello $_".say };

$action.( 'fluca1978' );
&$action( 'fluca1978' );
```
<br/>
<br/>

# Conclusions

Clearly Raku, being much more strict on types, as well as younger than Perl, has some advantages in the usage of inner code blocks. In either case, once you got used to code blocks, you will use them everywhere!

```
