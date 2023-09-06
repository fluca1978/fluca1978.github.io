---
layout: post
title:  "Dynamic Method Dispatching in Perl"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
Perl is a language rich in features, and one that I seldomly use is the dynamic method dispatching.

# Dynamic Method Dispatching in Perl

Perl is a very dynamic and flexible language, we all know that!
<br/>
In this post I present a feature that I seldomly use, that is the dynamic method dispatching.
<br/>
I tend not to use, nor abuse, this feature because it can become very quickly a root for errors and reduces the code readability. Moreover, having **worked very hard with reflection and dynamic methods** (see my PhD thesis for an idea), it reminds me very hard times!
<br/>
However, there are some edge cases where this Perl feature can save you a lot of code, and in this article I try to simply demonstrate it.

## The Scenario

Imagine you want to build a `Person` class (e.g., using `Moo`, but that does not matter at all), and you want to give to the `Person` a set of capabilities that are very similar *in shape*. For example, a `Person` can have a list of *skills* and `achievements`: both are *lists* (arrays), no matter if you use a class to represent each of them or not.
<br/>
In this situation, you will quickly find yourself in writing code that, for example, count and stringify the above class properties.
<br/>
A simple application can run as the following:

<br/>
<br/>
```perl
package Person;
use Moo;

has name    	 => ( is => 'ro' );
has skills  	 => ( is => 'rw', default => sub { [] } );
has achievements => ( is => 'rw', default => sub { [] } );




package main;

my $luca = Person->new( name => 'Luca' );
push $luca->skills->@*, $_ for ( qw/ Perl Raku Java PHP C / );
push $luca->achievements->@*, $_ for ( qw/ degree PhD / );


# print out the person details
say 'The person ' . $luca->name . ' has: ';
say "\t" . scalar( $luca->skills->@* ) . " skills ";
say "\t" . join( ', ', $luca->skills->@* );
say ' and has: ';
say "\t" . scalar( $luca->achievements->@* ) . " achievements ";
say "\t" . join( ', ', $luca->achievements->@* );

```
<br/>
<br/>


The output of the application is very straighforward:

<br/>
<br/>
```shell
% perl test.pl
The person Luca has:
        5 skills
        Perl, Raku, Java, PHP, C
 and has:
        2 achievements
        degree, PhD

```
<br/>
<br/>


Did you spot the problem? Well, there is no problem so far, but sooner or later, for sake of readability, you will provide some class methods to help the data retrieval. The class and the application therefore could be modified as follows:


<br/>
<br/>
```perl
use v5.38;

package Person;
use Moo;

has name    	 => ( is => 'ro' );
has skills  	 => ( is => 'rw', default => sub { [] } );
has achievements => ( is => 'rw', default => sub { [] } );

sub count_skills {
    my ( $self ) = @_;
    scalar $self->skills->@*;
}

sub count_achievements {
    my ( $self ) = @_;
    scalar $self->achievements->@*;
}

sub list_skills {
    my ( $self ) = @_;
    join ', ', $self->skills->@*;
}

sub list_achievements {
    my ( $self ) = @_;
    join ', ', $self->achievements->@*;
}


package main;

my $luca = Person->new( name => 'Luca' );
push $luca->skills->@*, $_ for ( qw/ Perl Raku Java PHP C / );
push $luca->achievements->@*, $_ for ( qw/ degree PhD / );

say 'The person ' . $luca->name . ' has: ';
say "\t" . $luca->count_skills . " skills ";
say "\t" . $luca->list_skills;
say ' and has: ';
say "\t" . $luca->count_achievements . " achievements ";
say "\t" . $luca->list_achievements;

```
<br/>
<br/>

The methods `count_skills`, `count_achievements`, `list_skills`, `list_achievements` help improving the usage of your own class from the outside, but require you to write more code within the class.
<br/>
Do you see the problem now?
<br/>
*Every time you add a new property to the class, you need to implement a few utility/helper methods*, and this is annoying!


## Dynamic Dispatch to the Rescue

Being Perl, it is possible to solve the problem of adding new properties to the class without requiring new methods thanks to the dynamic method dispatch.
The idea is to write only the *template* methods, that can work on pretty much every list property of the class, and then use those methods to dynamically call the property methods on the class. This of course requires that (i) the properties have similar behavior and (ii) the counting, stringification, and in general the methods can apply the same logic to all the above properties.
<br/>
The class is modified as follows:


<br/>
<br/>
```perl
use v5.38;

package Person;
use Moo;

has name    	 => ( is => 'ro' );
has skills  	 => ( is => 'rw', default => sub { [] } );
has achievements => ( is => 'rw', default => sub { [] } );

sub count {
    my ( $self, $what ) = @_;
    scalar $self->$what()->@*;
}

sub list {
    my ( $self, $what ) = @_;
    join ', ', $self->$what()->@*;
}


package main;

my $luca = Person->new( name => 'Luca' );
push $luca->skills->@*, $_ for ( qw/ Perl Raku Java PHP C / );
push $luca->achievements->@*, $_ for ( qw/ degree PhD / );

say 'The person ' . $luca->name . ' has: ';
say "\t" . $luca->count( 'skills' ) . " skills ";
say "\t" . $luca->list( 'skills' );
say ' and has: ';
say "\t" . $luca->count( 'achievements' ) . " achievements ";
say "\t" . $luca->list( 'achievements' )
```
<br/>
<br/>


Let's start from the application first: as you can see the methods now accept a string value (e.g., `'skills'`) that is used to discriminate between the different class properties. The methods, now `count` and `list` operate on the `$what()` class method, that is in turn resolved into a class method.
<br/>
For instance, `$luca->count( 'skills' );` is translated into `scalar $self->$what()->@*` that in turn is translated into `scalar $self->skills()->@*`. The trick here is that `$what` is used to store **the method name, that in turn is activated by appending `()` after the variable evaluation** (and note that the parentheses are not even mandatory!).

<br/>
<br/>

This provides room also for another improvement in the code that exploits your class:

<br/>
<br/>
```perl
package main;

my $luca = Person->new( name => 'Luca' );
push $luca->skills->@*, $_ for ( qw/ Perl Raku Java PHP C / );
push $luca->achievements->@*, $_ for ( qw/ degree PhD / );

say 'The person ' . $luca->name . ' has: ';
for ( qw/ skills achievements / ) {
    say "\t" . $luca->count( $_ ) . " $_";
    say "\t" . $luca->list( $_ ) . " $_";
}


```
<br/>
<br/>


In other words, you can loop over a set of properties (by their names) and get out the data from the generic and dynamic methods.
<br/>
This is, by far, the only way I use this approach: when I need to produce long reports over datasets (e.g., spredsheets), I tend to implement generic method that wrap dynamic dispatch and loop over all the properties I want to extract.


# Conclusions

Perl is really a dynamic and flexible language, that allows you to shrink your code in a very clever way.
<br/>
Less code, generally means less errors and, most notably, more fun!
