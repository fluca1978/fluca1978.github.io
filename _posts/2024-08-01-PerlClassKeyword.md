---
layout: post
title:  "The new Perl __CLASS__ keyword"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
The new `__CLASS__` keyword added in Perl v5.40.

# The new Perl __CLASS__ keyword

Since version `5.38` Perl has a **`class`** keyword that, among other related keywords, allow users to define classes in a `Moose` like approach.

The keywords are acting as "glue" over the package subsystem to declare classes in the Perl way (i.e., as the old days of `bless`).

And now, with the advent of Perl `v5.40`, a new `__CLASS__` keyword has been added. The keyword works pretty much like `__PACKAGE__`, but is consistent with the class the code lives in, with respect to the subclasses.

In order to demonstrate this, consider the following example:

<br/>
<br/>
```perl
use v5.40;
use feature 'class';

class Person {
    field $name    :param;
    field $surname :param;

    method present_yourself {
    	return "I'm a " . __PACKAGE__ .', ' . $surname . " " . $name;
    }
}

class PerlProgrammer :isa( Person ) { }

```
<br/>
<br/>

The class `PerlProgrammer` is a subclass of `Person`, hence inherits attributes and methods.

Now, consider the following snippet of code:

<br/>
<br/>
```perl
my $luca = PerlProgrammer->new( name => 'Luca', surname => 'Ferrari' );
say $luca->present_yourself;
say 'He is a Perl Programmer!' if $luca->isa( q/PerlProgrammer/ );

```
<br/>
<br/>

When executed, the above prints:

<br/>
<br/>
```shell
I'm a Person, Ferrari Luca
He is a Perl Programmer!
```
<br/>
<br/>

So, while `isa` correctly understands that `$luca` is holding a subclass reference, the `__PACKAGE__` keyword does not because such keyword gets the value of the *namespace* the code lives within. Let's adjust this to the new `__CLASS__` keyword:

<br/>
<br/>
```perl
use v5.40;
use feature 'class';

class Person {
    field $name    :param;
    field $surname :param;

    method present_yourself {
    	return "I'm a " . __CLASS__ .', ' . $surname . " " . $name;
    }
}

class PerlProgrammer :isa( Person ) { }
```
<br/>
<br/>

and run the same code snippet again, this time getting the following result:

<br/>
<br/>
```shell
I'm a PerlProgrammer, Ferrari Luca
He is a Perl Programmer!

```
<br/>
<br/>

So `__CLASS__` is correctly finding out the dynamic value of the *namespace* the code lives in.
This clearly is useful for introspection and dynamic management of intra-inheritance code.
