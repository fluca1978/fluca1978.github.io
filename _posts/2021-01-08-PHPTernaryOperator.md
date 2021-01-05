---
layout: post
title:  "PHP Ternary Operator: Unparenthesized `a ? b : c ? d : e` is deprecated"
author: Luca Ferrari
tags:
- php
- raku
- perl
permalink: /:year/:month/:day/:title.html
---
Another strange thing in PHP-land...

PHP Ternary Operator: Unparenthesized `a ? b : c ? d : e` is deprecated
---

Today a colleague of mine pointed out a problem on a PHP application:
<br/>
```
PHP Error : Unparenthesized `a ? b : c ? d : e` is deprecated. 
Use either `(a ? b : c) ? d : e` or `a ? b : (c ? d : e)` 
```
<br/>

Thankfully, the error was indicating also the line number, so it was a 30-seconds fix.
<br/>
But what was the problem?
<br/>
I decided to dig a little more, and find out that PHP `7.4` (and may be something between `7.1` and `7.4`, I cannot confirm) deprecates the usage of *cascading ternary operators**.
<br/>
**I do love ternary operator**, it simply allows me to compact `rvalue`s in pretty much any language I use (and in Perl also to `lvalue` it, but hey, not many languages are great as Perl is...).
<br/>
I do love ternary operator so much that I started also to cascade them, that means to use a ternary operator as a branch of another ternary operator. So far so good, unless I discovered that PHP does not support very well this feature.
<br/>
Or better, it does support, but **right-to-left**, that not only is counterintuitive, but also **different from pretty much any other language**. The latter means that what you are expecting to work in any decent language is not going to work that way in PHP.
<br/>
Allow me to demonstrate with a quick example:

<br/>
<br/>
```perl
use v5.20;
my $a = 10;
my $b = 20;
my $c = 30;
my $d = 40;
my $e = 50;

say "With all set should print b = $b\n";
say $a ? $b : $c ? $d : $e;
```
<br/>
<br/>
What is the above going to print? 
<br/>
Let's discuss: `$a` is set, so the first branch is selected, therefore `$b` is printed.

<br/>
<br/>
```perl
% perl test.pl
With all set should print b = 20

20
```
<br/>
<br/>


The result is the same in Raku:


<br/>
<br/>
```shell
% raku test.p6
With all set should print b = 20

20

% cat test.p6
#!raku
my $a = 10;
my $b = 20;
my $c = 30;
my $d = 40;
my $e = 50;

say "With all set should print b = $b\n";
say $a ?? $b !! $c ?? $d !! $e;
```
<br/>
<br/>

So, the question is: what about PHP?
<br/>
Surprisingly, it works in a very different way:
<br/>
<br/>
```shell
% php test.php
With all set should print b = 20
40

<?php

$a = 10;
$b = 20;
$c = 30;
$d = 40;
$e = 50;

echo "With all set should print b = $b\n";
echo isset( $a ) ? $b : isset( $c ) ? $d : $e;
```
<br/>
<br/>


What is happening here? In PHP, due to the design of the parser, the ternary operator is executed from right to left, or better, the inner operator has the precedence and therefore it reads as `isset( $c ) ? $d : $e` and then the `isset( $a ) ? $b` branch is executed.
<br/>
Allow me to explain with another example:

<br/>
<br/>
```shell
% php test.php
With all set should print b = 20
20
 b = 21 d = 40 e = 50
 
% cat test.php
<?php

$a = 10;
$b = 20;
$c = 30;
$d = 40;
$e = 50;

echo "With all set should print b = $b\n";
echo isset( $a ) ? $b : ( isset( $c ) ? $d : $e );

isset( $a ) ? ++$b : ( isset( $c ) ? ++$d : ++$e );
echo "\n b = $b d = $d e = $e";

?>
```
<br/>
<br/>

As you can see, with parentheses the parser is able to understand that the first colon term is something apart from the outer expression, and therefore will be evaluated only when effectively needed.

<br/>
<br/>
```shell
% php test.php

 b = 21 d = 41 e = 50
```
<br/>
<br/>

And therefore **both true branches have been executed**, even if the `isset( $c )` should have been totally short-circuited.
<br/>
Awful!

# PHP 7.4 and possible solution

The above behavior is not a result of the `7.4` version, it has been there since day one!
<br/>
[Wikipedia](https://en.wikipedia.org/wiki/%3F:#PHP){:target="_blank"} states it loudly:

<br/>
<br/>
```
Due to an unfortunate design of the language grammar, 
the conditional operator in PHP is left associative 
in contrast to other languages [..]
```
<br/>
<br/>

PHP 7.4 has only added a deprecation to this *unwanted* behavior, in order to force you to write it down the right way, that is using parenthesis to explicitly tell the priority:


# Conclusions

I don't like PHP that much, and this even before I get to know the above dizzy feature.
I think it would have been better to remove that behavior when it was still possible, but I could be wrong.
<br/>
So the real conclusion is that **sometimes the parser can get a very hard time in trying to figure out what you are trying to do**. And this is true for every programming language.
