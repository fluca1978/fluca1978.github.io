---
layout: post
title:  "Perl 5.38 Class Support: a glance at"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
The fresh new release of Perl added support for OOP in a declarative way.

# Perl 5.38 Class Support: a glance at

The new release of Perl, namely **Perl v5.38** adds a declarative support for Object Oriented Programming.
<br/>
*Wait a minute: Perl 5 had OOP support since day one!*
<br/>
And that is true!
<br/>
What Perl v5.38 brings to the table is _declarative_ support, that means a new bunch of keywords to quickly and easily define classes, members and methods.

<br/>
<br/>
The new keywords introduces are:
- **`class`** defines a new class, accepts a block and, behind the hood, works as a `Package` declaration;
- **`method`** defines a subroutine lexically scoped to the class and hence bound to the object the class will define.
- **`field`** defines a public attribute, or member, of the class.


<br/>
<br/>
Quite frankly, we are much behind what OOP modules like `Moose`, `Moo` and alike do provide, but there is room for improvements, as in all things Perl, so I believe we will have very soon a more cleaner way of defining classes.


## A Simple Class Example

Let's see a very simple example:

<br/>
<br/>
```perl
use v5.38;

use feature 'class';
no warnings 'experimental::class';

class Person {
    field $name :param;
    field $surname :param;
    field $gender;

    # get a stringy description of
    # this person
    method who {
  	    return ( $gender =~ /^M(ale)?$/i ? 'Mr' : 'Ms' )
		        . " $name $surname";
    }

    ADJUST {
    	$gender //= 'M';
   }
}


my $luca = Person->new( name => 'Luca', surname => 'Ferrari' );
say $luca->who;


```
<br/>
<br/>



First of all, in order to use the new keywords you need to both enable version `v5.38` as well as the `class` feature.
<br/>
The `class` keyword defines the block that handles the class, and the variables are declared as normal variables in Perl, with the `field` declarator that makes them members of the ogoing object. The traits of each member define how the member will behave. So far, the only one supported is `:param` that defines that the value could be specified as a named pair within an anonymous hash at object construction time.
<br/>
And that leads to the fact that the *`class` will give you a constructor for free!*
<br/>
Variables declared with `field` are lexically scoped within the class block, as you can see in the method `who`, where their usage does not require any particular scoping. The `method` keyword defines a subroutine that is going to be used as an object method.
<br/>
The `class` keyword also provides the capability for an `ADJUST` special block that will be called during the construction, and can be used to optimize the object construction.

<br/>
In the end, you see how to use the class to create an object, initialize it and access methods.

<br/>
<br/>

Note how the code is cleaner, with respect to the usual usage of Perl OOP: `bless` is totally absent, but also there is no need for `$self` (or a reference like that). It is important to note that `class` and its constructor(s) do not provide a `bless` wrapper, rather a new in-language way of creating objects.


## What is What?

The usage of `ref` against an object will return, well, the class name. Therefore `ref $luca` will return `Person`, as you would expect.
The usage of `builtin::reftype` will return the `OBJECT` specifier. This is a little different from what happens with the *old way of blessing objects*.

## What is missing?

Not to be harsh, but a lot of things if you compare this to other OOP frameworks.
<br/>
Most notably, there is no support for accessors, but work is ongoing and in the next feature you will be probably able to trait a member indicating if it is accessible in read and/or write mode.
<br/>
Roles are still missing...


# Conclusions

Perl keeps improving at every release, and soon will have a complete and mature declarative OOP infrastructures.
In the meantine, keep reading the `perlclass` documentation!
