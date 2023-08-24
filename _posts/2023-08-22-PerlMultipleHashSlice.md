---
layout: post
title:  "Perl Multi Hash Slice"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
How to slice an hash (reference) with multiple keys.

# Perl Multi Hash Slice

While the syntax to slice an array in Perl is quite staighforward, I often miswrite the same concept when dealing with hashes.
Perl being Perl, it is of course possible to slice hashes even with multiple keys, but when you handle a reference to the hash the sigils get weird, in particular when using *post-dereferencing*.

## A Simple Hash Slicing (without references)

If all you have is an hash, slicing it is as simple as it may sound:

<br/>
<br/>
```perl
my %data = (
	    username => 'luca',
	    password => 'secret',
	    url      => 'https://fluca1978.github.io',
	   );

my ( $web, $who ) = @data{ qw/ username url / };
say "You can reach $who at $web";

```
<br/>
<br/>

The only thing to remember here is to change the sigil of the hash from `%` to `@` since you want to retrieve more than one element at the same time.

## A Not Working Hash Slicing (with references)

If you handle a reference to the hash, things get a little more complicated. The following is not working:

<br/>
<br/>
```perl
my $data = {
	   username => 'luca',
	   password => 'secret',
	   url      => 'https://fluca1978.github.io',
	  };

my ( $web, $who ) = $data->{ qw/ username url / };
say "You can reach $who at $web";

```
<br/>
<br/>

As well as any combination you can think of is not working at all:
- `$data->{ qw // }->@*` is not working because you are not going to have an array reference;
- `@$data->{ qw// }` ditto.


## Thw Working Hash Slicing (with references)

The working solution is to *dereference the hash, tell you are going to extract a list, and give the keys*:

<br/>
<br/>
```perl
my $data = {
	   username => 'luca',
	   password => 'secret',
	   url      => 'https://fluca1978.github.io',
	  };

my ( $web, $who ) = $data->@{ qw/ username url / };
say "You can reach $who at $web";

```
<br/>
<br/>

Therefore:
- deference `$data->`;
- ask for a list with `@`;
- give the keys to the hash with `{ qw // }`.

This is *a very odd syntax* according to me, since you are using something that is a list `@` with hash curlies.


## Understanding what is doing what

It is possible to use `Deparse` to understand how Perl is going to handle the whole situation:

<br/>
<br/>
```shell
% perl -MO=Deparse test.pl
sub BEGIN {
    require v5.20;
    ()
}
use strict;
no feature ':all';
use feature ':5.16';
my $data = {'username', 'luca', 'password', 'secret', 'url', 'https://fluca1978.github.io'};
my($web, $who) = @$data{'username', 'url'};
say "You can reach $who at $web";
test.pl syntax OK

```
<br/>
<br/>

As you can see, the `$data->@{ .. }` has been translated to `@$data{ .. }`.

## Multidimensional Hashes

Thanks to `Deparse` it is now possible to understand why `$data->{ qw// }` does not work:

<br/>
<br/>
```shell
 % perl -MO=Deparse test.pl
sub BEGIN {
    require v5.20;
    ()
}
use strict;
no feature ':all';
use feature ':5.16';
my $data = {'username', 'luca', 'password', 'secret', 'url', 'https://fluca1978.github.io'};
my($web, $who) = $$data{join $;, 'username', 'url'};   # $data->{ qw/ username url / }
say "You can reach $who at $web";
test.pl syntax OK

```
<br/>
<br/>

As you can see, there is now an explicit call to `join` in order to construct a kind of *super-key* for the hash.
The idea is this: since you need to extract a key, you combine a list of keys into the super key and return that valye. This is ancient logic that comes directly from Perl 4, when multidimensional stuff was not available. It can be checked (and enabled) via the `feature` pragma:

<br/>
<br/>
```shell
% perldoc feature

 The 'multidimensional' feature
    This feature enables multidimensional array emulation, a perl 4 (or
    earlier) feature that was used to emulate multidimensional arrays with
    hashes. This works by converting code like $foo{$x, $y} into
    $foo{join($;, $x, $y)}. It is enabled by default, but can be turned off
    to disable multidimensional array emulation.


```
<br/>
<br/>

Since *5.36* the multidimensional hashes are disabled by default, since you can now nest hashes as you like, therefore the system will complain:

<br/>
<br/>
```shell
% perl test.pl
Multidimensional hash lookup is disabled at test.pl line 39, near "qw/ username url / }"

```
<br/>
<br/>

You can enable it by means of adding `use feature qw/ multidimensional /;` but that is not going to work as you can expect:

<br/>
<br/>
```perl
use v5.36;
use feature qw/multidimensional/;

my $data = {
	   username =>  'luca',
	    password => 'secret',
	    url      => 'https://fluca1978.github.io',
	    usernameurl => 'Not what you are looking for!',

	  };

my ( $web, $who ) = $data->{ qw/ username url / };
say "You can reach $who at $web";

```
<br/>
<br/>


Why is the above not working?
<br/>
Well, `qw` returns a list, not a single key, while `multidimensional` wants to build a key out of list of them, therefore there is no match!
<br/>
In other words, forget the multidimensional hash stuff and nest hashes as you want.


# Conclusions

Hash slices can be a little tricky when dealing with references, and according to me the `->@{ }` syntax is not as obvious as it may sound, but it is just a matter of knowing what you are handling.
Perl being Perl, this is fun stuff, isn't it?
