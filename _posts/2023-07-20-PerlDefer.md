---
layout: post
title:  "Perl defer blocks"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
A new phaser in Perl!

# Perl defer blocks

Perl recently (well, not so recently), introduced a new experimental feature called **`defer` blocks**.
<br/>
A `defer` block is just a block that is executed as a last resort when the lexical scope of the containing block expires. To some extent, it is a phaser block that is run at the very last time.
<br/>
A simple example can prove its usage:

<br/>
<br/>
```perl
use feature 'defer';
no warnings;

my $sub = sub {
    defer { say "... and see you later!" }

    say "Hello $_[0]"  for ( 1 .. 3 );

};


$sub->( q/Luca/ );


```
<br/>
<br/>

In the above code, the subroutine declares a `defer` block that will run at the very end of the scope of the subroutine block itself.
Once the subroutine is called, its body block is executed, as usual. Once the subroutine is going to complete, and thus expiring its lexical scope, the `defer` block is executed. Therefore, the output is:

<br/>
<br/>
```shell
Hello Luca
Hello Luca
Hello Luca
... and see you later!

```
<br/>
<br/>

Clearly, this implies that the `defer` block is executed as soon as the code block exits, no matter why it is exiting:

<br/>
<br/>
```perl
use feature 'defer';
no warnings;

my $sub = sub {
    defer { say "... and see you later!" };

    for ( 1 .. 5 ) {
		say "Hello $_[0]";
		return if $_ % 2 == 0;
    }

};


$sub->( q/Luca/ );

```
<br/>
<br/>

In the above, the function returns after a couple of iterations, not concluding the `for` loop, but the `defer` block is executed at last as expected:

<br/>
<br/>
```shell
Hello Luca
Hello Luca
... and see you later!

```
<br/>
<br/>


After all, the rule is simple: **a `defer` block is _always_ executed at last**, even if an exception occurs:

<br/>
<br/>
```perl
use feature 'defer';
no warnings;

my $sub = sub {
    defer { say "... and see you later!" };

    for ( 1 .. 5 ) {
	say "Hello $_[0]";
	die "Argh!";
    }

};


$sub->( q/Luca/ );


```
<br/>
<br/>


The above code produces the following output:

<br/>
<br/>
```shell
Hello Luca
Argh! at /home/luca/tmp/test.pl line 13.
... and see you later!

```
<br/>
<br/>



## `defer` or `continue` ?

Resist to the temptation of using `defer` as a phaser in loops, since this is not going to produce what you want (or maybe it will produce it, depending on what you want, ehm):


<br/>
<br/>
```perl
for ( 1 .. 3 ) {
    say "Counting $_";
    defer { say "At last the defer block!" }
}

```
<br/>
<br/>

will produce:

<br/>
<br/>
```shell
Counting 1
At last the defer block!
Counting 2
At last the defer block!
Counting 3
At last the defer block!

```
<br/>
<br/>

In fact, since every time the loop completes an iteration its code block goes out of scope, the `defer` block is executed at every iteration.
Therefore, it looks like `defer` behaves exactly as a `conitnue` block, but there is an important difference: being defined into the scope of the containing block, the `defer` block has access to lexical variables, while the `continue` block does not!


# Conclusions

`defer` is another piece of art to make great Perl programs!
<br/>
It allows to define a *finally-like behavior* even when no exceptions are in charge, and can be used similarly to a `continue` block within loops, but with some extra powers!
