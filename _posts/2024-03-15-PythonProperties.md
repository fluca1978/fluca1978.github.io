---
layout: post
title:  "Python Properties Explained Simply"
author: Luca Ferrari
tags:
- python
- perl
permalink: /:year/:month/:day/:title.html
---
What is the aim of `@property`?

# Python Properties Explained Simply

One thing that, quite frankly, I found confusing about Python was the `@property` decorator: it seems no one is able to explain it in simple terms, so here it is my chance to explain it.

I assume the reader already knows what is a *decorator*, that in short terms is a function executed around another one.

## The Java-bean way: getters and setters

Let's start by producing a class with standard getters and setters, as Java would like:

<br/>
<br/>
```python
class User:
    def __init__( self, uname, name ):
        self._username = uname
        self._name     = name

    def get_username(self):
        return self._username

    def get_name(self):
        return self._name

    def set_name( self, name ):
        self._name = name

```
<br/>
<br/>

The class `User` has two **private** attributes named `_username` and `_name`, and there is the possibility to get their values calling a `get_` method and set the name calling `set_name()` method.

Therefore, the class can be used as:

<br/>
<br/>
```python
u = User( 'fluca1978', 'Luca Ferrari' )
print( "Username is " + u.get_username() )
print( "Name is " + u.get_name() )

```
<br/>
<br/>


## Removing the need for accessor methods: using public attributes

If you want to remove the need for the `get_` and `set_` methods, you can clearly define the class attributes as **public**.
This is somehow a poor Object Oriented Design, since it exposes your implementation details to the rest of the world, making it really hard to change your own library someday in the future.

<br/>
<br/>
```python
class User:
    def __init__( self, uname, name ):
        self.name     = name
        self.username = uname

```
<br/>
<br/>


Now it is possible to use the attributes of the class without having to call a method, that means without the need for the parens:

<br/>
<br/>
```python
u = User( 'fluca1978', 'Luca Ferrari' )
print( "Username is " + u.username )
print( "Name is " + u.name )

```
<br/>
<br/>


There is however a possible problem of this approach: entities outside your object can modify its state directly.

<br/>
<br/>
```python
u = User( 'fluca1978', 'Luca Ferrari' )
u.username = 'jikesrvm'
u.name     = 'Ferrari Luca'
print( "Username is " + u.username )
print( "Name is " + u.name )

```
<br/>
<br/>

Probably this is not what you want.


## Towards a *property-like* approach

Having to type `get_` in front of a method that returns a value is boring, let's change the methods using the *Qt's approach* instead of the Java one: the getter method will be named as the attribute, keeping the `set_` in front of only mutators.

<br/>
<br/>
```python
class User:
    def __init__( self, uname, name ):
        self._username = uname
        self._name     = name

    def username(self):
        return self._username


    def name(self):
        return self._name

    def set_name( self, name ):
        self._name = name

u = User( 'fluca1978', 'Luca Ferrari' )
print( "Username is " + u.username() )
print( "Name is " + u.name() )

```
<br/>
<br/>

While this is surely a more readable way, it still requires parens to let Python to understand we are calling a method.


## Using `@property`

The `@property` decorator marks a *method* for usage as **it was representing a plain public attribute**.
Let's see this in action:

<br/>
<br/>
```python
class User:
    def __init__( self, uname, name ):
        self._username = uname
        self._name     = name

    @property
    def username(self):
        return self._username

    @property
    def name(self):
        return self._name

    def set_name( self, name ):
        self._name = name

u = User( 'fluca1978', 'Luca Ferrari' )
print( "Username is " + u.username )
print( "Name is " + u.name )

```
<br/>
<br/>

The important thing to note here is **that there are no parens as in a method call**, even if the method call is happening.
Is it really?

<br/>
<br/>
```python
class User:
    def __init__( self, uname, name ):
        self._username = uname
        self._name     = name

    @property
    def username(self):
        print( "\tusername() called")
        return self._username

    @property
    def name(self):
        print( "\tname() called" )
        return self._name

    def set_name( self, name ):
        self._name = name


```
<br/>
<br/>

that produces the output:

<br/>
<br/>
```shell
% python3 test.py
        username() called
Username is fluca1978
        name() called
Name is Luca Ferrari

```
<br/>
<br/>

So the methods are effectively called, via the decorator, that lets Python to manage the attributes (non-existent!) as public members.

### `@property` side effects

There are two important concepts to keep in mind when adding `@property`:
- by default `@property` works only for a *readable* attribute, that means you cannot assign to an attribute a value:

<br/>
<br/>
```python
 u.name = 'Luca'
    ^^^^^^
AttributeError: property 'name' of 'User' object has no setter

```
<br/>
<br/>

- the method *wrapped* as a `@property` is no more directly callable, since it does not exist anymore (the wrapping method provided by the decorator exist):

<br/>
<br/>
```python
print( "Name is " + u.name() )
                   ^^^^^^^^
TypeError: 'str' object is not callable

```
<br/>
<br/>


## Using `@property.set` to modify a property

There is another decorator, `@property.set`, that can be used to specify which method acts as a setter for a property:

<br/>
<br/>
```python
class User:
    def __init__( self, uname, name ):
        self._username = uname
        self._name     = name

    @property
    def username(self):
        print( "\tusername() called")
        return self._username

    @property
    def name(self):
        print( "\tname() called" )
        return self._name

    @name.setter
    def name( self, name ):
        self._name = name

u = User( 'fluca1978', 'Luca Ferrari' )
u.name = 'jikervm'
print( "Username is " + u.username )
print( "Name is " + u.name )

```
<br/>
<br/>

There are a few changes to the code:
1) the `set_name` method has changed to `name`;
2) the setter has been decorated with `@name.setter` where `name` is the name of the property it refers to, in this case `name`.

In this way, `u.name` can be used as an *rvalue* (via `@property`) or *lvalue* (via `@name.setter`).


## What about Perl?

Well, Perl already allows for a more *attribute* like approach:

<br/>
<br/>
```perl
package Person;

sub new {
    bless {}, shift;
}

sub name {
    my ( $self, $name ) = @_;
    $self->{ name } = $name if ( $name );
    return $self->{ name };
}

```
<br/>
<br/>

The method `name`, when called without arguments, works as a *getter*, while then called with arguments acts as a *setter*:

<br/>
<br/>
```perl
my $person = Person->new;
$person->name( 'Luca' ); # as lvalue
say $person->name;       # as rvalue

```
<br/>
<br/>


But there's more: Perl has the (depcrecated) `lvalue` traits that does what Python tries to achieve with `@property`:

<br/>
<br/>
```perl
package Person;

sub new {
    bless {}, shift;
}

sub name : lvalue {
    return shift->{ name };
}

```
<br/>
<br/>

With `lvalue` Perl knows that the method `name` can act **also as an `lvalue`**, therefore when Perl sees `name` as an `lvalue` invokes the method with the *rvalue* expression:

<br/>
<br/>
```perl
my $person = Person->new;
$person->name = 'Luca';   # as lvalue
 say $person->name;       # as rvalue

```
<br/>
<br/>


# Conclusions

Python's `@property` decorator is surely useful to improve readability of the code, even if it seems to me too much work for getting a method to behave as an attribute.
On the other side, since Perl doesn't require parens if unnecessary, a *read-only like property* comes for free with a correct method name!
