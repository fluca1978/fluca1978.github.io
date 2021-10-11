---
layout: post
title:  "PHP Warning: \"continue\" targeting switch is equivalent to \"break\". Did you mean to use \"continue 2\"?" 
author: Luca Ferrari
tags:
- php
- programming
permalink: /:year/:month/:day/:title.html
---
PHP is not a sane language, according to me!

# PHP Warning: "continue"  targeting switch is equivalent to "break". Did you mean to use "continue 2"?

A colleague of mine pointed me to this strange warning that started to appear in a server logs.
<br/>
I'm not a PHP expert, but I use since twelve years (at least), and I know it carries on old behaviours due to a not so solid implementation of the language itself, and apparently, this is another one.
<br/>
Here I'm going to discuss what the above warning referes to, but the important thing to note is that **the above is a warning, and does not mean your code is going to stop working**, on the other hand it advices that **in future PHP releases, something may change and your code could break!**

## `break` and `continue`

In most *sane* languages, in particular those that derive from C, the two keywords do not have the same meaning:
- `break` means "exit from the current loop *immediatly*";
- `continue` means "stop the current iteration and restart the loop from the very next one".

<br/>
Therefore, `break` means the loop must be terminated, `continue` means that the current iteration must be terminated, but the next one can continue.
<br/>
The problem here arises from the fact that `break` can be used into other structures, most notably the `switch` one: even in such context, `break` means "exit from the `switch` right now". In other words, you can think of `break` as "*exit from the code block immediatly*", without any regard the code block is a loop or not.
<br/>
In fact, C and derived languages have a `switch` implementation that is not the equivalent of an `if-elsif-elsif...` branching as it is often taught: the `switch` means "jump into the first match and go on from there", where the `break` is used to exit the matching branch as soon as it finishes.
<br/>
Raku, for example, provides a `given when` construct that is equivalent to a C `switch` with "automatic" `break` on each branch (and a lot more power).

## Is `switch` a loop?

Of course `switch` is not a looping construct, however in  [PHP the `switch` **is considered a looping** block](https://www.php.net/manual/en/control-structures.continue.php){:target="_blank"}:

<br/>
<br/>
```
Note: In PHP the switch statement is considered a looping structure for the purposes of continue. continue behaves like break (when no arguments are passed) but will raise a warning as this is likely to be a mistake. If a switch is inside a loop, continue 2 will continue with the next iteration of the outer loop. 
```
<br/>
<br/>

So, not only the `switch` is considered and implemented as a looping block, but PHP considers a `break` a mistake inside a `switch`. Why? Because in a looping structure you probably want to `continue`, not to `break**.
<br/>
**And this is why I don't believe PHP is a sane language:** reading the code you are going to find a keyword `continue`, that to my poor brain means, well, "continue", that however is going to be implemented as a "break".
<br/>
<br/>
But it does not ends here: if `continue` has a numeric argument, it will try to restart the loop outer of the specified number of levels. The innermost loop is at level 1, the outer loop is at level 2, and so on. Here's why the system suggest you to write a `continu 2`, that will make the program flow to continue to the outer loop iteration.

## An Example

Let's start with a looping example: nest three level of `foreach`:

<br/>
<br/>
```php
<?php
$array = array( 1, 2, 3, 4 );

foreach( $array as $big_outer ) {
    foreach( $array as $outer ) {
        foreach( $array as $inner ) {
            print "$big_outer $outer-$inner \n";
            continue 3;
        }
    }
}

?>
```
<br/>
<br/>

The above piece of code produces the following output, because at each iteration the loop starts over from the level 3, that is the outermost one:

<br/>
<br/>
```shell
% php test.php
1 1-1 
2 1-1 
3 1-1 
4 1-1 
```
<br/>
<br/>

If the `continue 3` is replaced with a `continue 2` the output changes, because only the innermost loop is interrupted:

<br/>
<br/>
```shell
% php test.php
1 1-1 
1 2-1 
1 3-1 
1 4-1 
2 1-1 
2 2-1 
2 3-1 
2 4-1 
3 1-1 
3 2-1 
3 3-1 
3 4-1 
4 1-1 
4 2-1 
4 3-1 
4 4-1 
```
<br/>
<br/>

Let's see what happens with a `switch` within a loop:


<br/>
<br/>
```php
<?php
$array = array( 1, 2, 3, 4 );

foreach( $array as $outer ) {
    print "Loop $outer \n";

    foreach( $array as $current ) {
        switch( $current ) {
        case 2: print "Two! \n";
            continue 2;

        case 4:
            print "Four! \n";
            continue 2;
        }
    }
}

?>

```
<br/>
<br/>

that produces the following output:

<br/>
<br/>
```shell
% php test.php
Loop 1 
Two! 
Four! 
Loop 2 
Two! 
Four! 
Loop 3 
Two! 
Four! 
Loop 4 
Two! 
Four! 

```
<br/>
<br/>


## `continue` with an argument is prone to errors (?)

The `continue` keyword cannot accept a number of levels higher than those the control flow detects:

<br/>
<br/>
```php
<?php
$array = array( 1, 2, 3, 4 );

foreach( $array as $outer ) {
    print "Loop $outer \n";

    foreach( $array as $current ) {
        switch( $current ) {
        case 2: print "Two! \n";
            continue 10;

        case 4:
            print "Four! \n";
            continue 10;
        }
    }
}

?>

```
<br/>
<br/>

produces the runtime error `PHP Fatal error:  Cannot 'continue' 10 levels`.
<br/>
This could be a problem because, when you refactor the code, you need to inspect the *deep* of each level of iteration and adjuct all the `continue` keywords. On the other hand, if you increase the level of nesting, you could mistakenly leave a lower argument to `continue` that does not produce a loop interruption.


# Conclusions

I don't like PHP very much, and the presence of these *odd behaviours*, like the `continue` within `switct` makes it a lot less readable that what I would like it to be.
<br/>
This does not mean that the language is bad, or it cannot work; it simply means it can be difficult to embrace PHP when you are used to other languages, even those that are similar by syntax.
