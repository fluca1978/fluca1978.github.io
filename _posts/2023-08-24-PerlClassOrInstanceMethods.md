---
layout: post
title:  "Perl OOP: using instance or class methods in an interchangeable way"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
Perl is a very dynamic and flexible language, that allows you to write a single method and use it as an instance or class one.

# Perl OOP: using instance or class methods in an interchangeable way

Almost twelve years ago I got a *job interview* and the interviewer asked me why I liked Perl.
Back in those days, my Perl knowledge was really limited, but I answered without any doubt saying that **Perl was fantastic because it allows, pretty much always, to program in an OOP manner or a procedural manner and the same library can be used in both ways depending on the client/user preferences**.

<br/>
This short article, tries to clearify my feelings and that old sentence.
The idea is that a method (instance sub) can be used in an OOP way and as a class method. For those who are familiar with Java, it means that *the same `sub` can act as a method and a `static` function*.

## A simple example

Let's build a very simple class, named `Person`, that stores the name and the username of a person. Then implement a method, named `equals` that checks if two instances of a `Person` are the same. To keep things simple, assume that two `Person`s are the same if the username matches, without any regard to the name.
<br/>
In the following there is a simple implementation that uses `Moo` as a shortcut to declarative classes. Please note that it does not matter how you build your classes: for the purpose of this article the method usage remains the same.

<br/>
<br/>
```perl
package Person;

use Moo;

has name       => ( is => 'ro' );
has username   => ( is => 'ro' );


sub equals {
    my ( $self, $comparing ) = @_;

    return undef if ( ! $self );  # only for class method !
    return undef if ( ! $comparing || ! $comparing->isa( ref $self ) );

    $self->username eq $comparing->username;
}

```
<br/>
<br/>

The `equals` method is the one that can behave at the same time as an instance method or a class subroutine.
The idea is that the method accepts two object references, `$self` (the usual *myself*) and the `$comparing` reference that is the object to compare against. The return value of the method, the last line, is the comparison of the `username` field.
<br/>
Note that the method will return `undef` (false) in the case the `$comparing` object is not an instance of the class itself. Similarly, it will return `undef` in the case `$self` is empty, which will never happen in the OOP style since `->` requires a valid object to call a method on. However, in the case the method is used as a class sub, it can happen that even `$self` is not an object.
<br/>
Having defined the method as above, it is possible to use it as:
- `$a->equals( $b )` where `$a` will become `$self` and `$b` will become `$comparing`. This is the **OOP method call**;
- `Person::equals( $a, $b )` where `$a` will become `$self` and `$b` will become `$comparing`. This is the **class function call**, and in this case `$a` could be empty.

<br/>
Why is this supposed to work?
<br/>
In the `->` call, the arrow operator passes its left argument as first parameter to the sub, i.e., to `$self`. This is a well known OOP way of doing method calls and is used also in Java and C++ to define `this`.
In the `::` call, Perl invokes a sub belonging to a package without adding any new parameter in front of the argument list, therefore providing only the two explicit arguments.


## Example at work

It is very simple to prove how the above class works:

<br/>
<br/>
```perl
package main;


my $luca = Person->new( name      => 'Luca Ferrari',
			username  => 'fluca1978' );

my $personA = Person->new( name     => 'Luca F.',
			   username => 'fluca1978' );

my $personB = Person->new( name     => 'Sofia',
			   username => 'blackcat' );

# OOP usage
say $luca->name    , " vs ", $personA->name, " = " , ( $luca->equals( $personA ) ? 'SAME!' : 'different people' );
say $luca->name    , " vs ", $personB->name, " = " , ( $luca->equals( $personB ) ? 'SAME!' : 'different people' );
say $personB->name , " vs ", $personA->name, " = " , ( $personB->equals( $personA ) ? 'SAME!' : 'different people' );

# class usage
say $luca->name    , " vs ", $personA->name, " = " , ( Person::equals( $luca, $personA ) ? 'SAME!' : 'different people' );
say $luca->name    , " vs ", $personB->name, " = " , ( Person::equals( $luca, $personB ) ? 'SAME!' : 'different people' );
say $personB->name , " vs ", $personA->name, " = " , ( Person::equals( $personA, $personB ) ? 'SAME!' : 'different people' );

```
<br/>
<br/>


The result of the above application is:

<br/>
<br/>
```shell
Luca Ferrari vs Luca F. = SAME!
Luca Ferrari vs Sofia = different people
Sofia vs Luca F. = different people

Luca Ferrari vs Luca F. = SAME!
Luca Ferrari vs Sofia = different people
Sofia vs Luca F. = different people

```
<br/>
<br/>


## The same class in another language (Java)

One advantage of Perl is that, as you will see, the code to write is far less. Assume we want to implement the same class `Person` in Java:

<br/>
<br/>
```java
public class Person {
   public String name;
   public String username;

   public boolean equals( Person comparing ) {
       return this.username.equals( comparing.username );
   }

   public static boolean equals( Person a, Person b ) {
       if ( a == null || b == null
	       || ! a instanceof Person || ! b instanceof Person )
	     return false;

      return a.equals( b );
   }
}

```
<br/>
<br/>


I've used two public members just to keep the code short.
The important thing to note in this implementation is that we need two `equals` methods: one for the object and one for the class.




## The operator overloading approach

There is more: it is possible to overload operators for a class, so that Perl will use the method when needed.
For example, instead of `$luca->equals( $person )` it is possible to write a more natural `$luca eq $person`.

<br/>
*WARNING: operator overloading is a powerful feature but require much more care about mixing different types and operators!*

<br/>

The class `Person` changes as follows:

<br/>
<br/>
```perl
package Person;

use overload
	'eq' => 'equals'
	;

use Moo;

has name       => ( is => 'ro' );
has username   => ( is => 'ro' );



sub equals {
    my ( $self, $comparing ) = @_;

    return 0 if ( ! defined $self );  # only for class method !
    return 0 if ( ! defined $comparing || ! $comparing->isa( ref $self ) );

    $self->username eq $comparing->username;
}

```
<br/>
<br/>

There are two main changes:
- introduction of the `overload` module usage, that defines that the `eq` operator will be *mapped* onto the `equals` class method;
- the `equals` method now checks explicitly about defined-ness of its argument, since overloading would require to define a way to check for boolean values (out of the scope of this article).


<br/>

having changed the class, it is now possible to write the code as follows:

<br/>
<br/>
```perl
say $luca->name    , " vs ", $personA->name, " = " , ( $luca eq $personA  ? 'SAME!' : 'different people' );
say $luca->name    , " vs ", $personB->name, " = " , ( $luca eq $personB  ? 'SAME!' : 'different people' );
say $personB->name , " vs ", $personA->name, " = " , ( $personB eq $personA  ? 'SAME!' : 'different people' );

```
<br/>
<br/>

which produces the same exact result as before.
<br/>
Just keep in mind that this approach can be risky if you start overloading too much operators and get clashes.





# Conclusions

Perl is extremely flexible and helps developers to meet their own taste in using libraries and modules.
While I believe that OOP is by far much more convenient than the subroutine approach, it does not mean that you cannot use both.
And also, this helps reducing the number of code to write.
