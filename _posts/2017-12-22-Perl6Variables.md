---
layout: post
title:  "Perl 6: more on variables"
author: Luca Ferrari
tags:
- perl6
- programming
permalink: /:year/:month/:day/:title.html
---
Perl 6 allows for a finest control over types variables can be assigned to.

# More on Perl 6 variables

There are several tricks when dealing with variables in Perl 6. In this article I will show how to deal with *binding* (what was in Perl 5 the *reference*) and how to mark a variable as a concrete container of values, types or both.


## Binding instead of assigning

When in Perl 6 there is an assignment, with the `=` operator, Perl 6 does not use a reference but does a copy of the right variable content into the left one. In simple words, you cannot modify the same object with different variables:

```perl
my Str $src = 'Hello World!';
my Str $dst = $src;
$dst = $dst.uc;

$src.say;
$dst.say;
```

that produces the following output pointing out that `$dst` **is a different object** than `$src`:

     Hello World!
     HELLO WORLD!

The same applies to complex data structures:

```perl
my @src = 1..10;
my @dst = @src;
@dst.push( 90, 91, 92 );

@src.say;
@dst.say;
```

that produces again a different output for the `@src` and `@dst` lists:

      [1 2 3 4 5 6 7 8 9 10]
      [1 2 3 4 5 6 7 8 9 10 90 91 92]

Of course, Perl 6 does allow to make two different variables pointing to the very same object, and that is called **binding**.
The operator is `[:=](https://docs.perl6.org/language/operators#index-entry-Binding_operator)` and it is important to note that **it is a compile time operator**.
Let's see that in action:

```perl
my Str $src = 'Hello World!';
my Str $dst := $src;
$dst = $dst.uc;

$src.say;
$dst.say;

my @src_array = 1..10;
my @dst_array := @src_array;
@dst_array.push( 90, 91, 92 );

@src_array.say;
@dst_array.say;
```

that produces the very same output for each couple of variables, meaning that modifying one of the couple modifies the real value:

       HELLO WORLD!
       HELLO WORLD!
       [1 2 3 4 5 6 7 8 9 10 90 91 92]
       [1 2 3 4 5 6 7 8 9 10 90 91 92]


There is also a read-only version of the binding operator, with a double colon `[::=](https://docs.perl6.org/language/operators#___top)`. Such operator prevents changes thru the bound variable, but at the time of writing [it has not yet been implemented](https://irclog.perlgeek.de/perl6/2017-12-22#i_15611575).


### Binding to literals makes constant-like behavior

In the case of a binding against a literal, the vaiable cannot change the value, so it *behaves as a constant*:

```perl
my Str $src := 'Hello World!';
$src = $src.uc;
```

produces a run-time error:

    Cannot assign to an immutable value
      in block <unit> at test.p6 line 65

The same applies to complex data structures, therefore:

```perl
my Str $src := 'Hello World!';
my @array := 1, 2, 3, $src;
@array.push( 'Goodbye!' );
```

that produces the run-time error:

    Cannot call 'push' on an immutable 'List'
      in block <unit> at test.p6 line 66

The same applies without any regard to the values in the list, that is **a bound list is made immutable by definition**.
Please note that the above is not a typo: binding an array produces a `List` and lists are immutable by definition.

## Sigil-less variables

There is a way to declare a *sigil-less* variable via the `\` operator. A sigil-less variables work as a normal variable, except that it has been bound to the right-hand value/container.
Therefore, `dst` becomes an alias of the `$src` variable, and the same for `dst_array`:

```perl
my Str $src = 'Hello World!';
my Str \dst = $src;
dst = dst.uc;

$src.say;
dst.say;

my @src_array = 1..10;
my \dst_array = @src_array;
dst_array.push( 90, 91, 92 );

@src_array.say;
dst_array.say;
```

that outputs the very same values for each couple of variables:

     HELLO WORLD!
     HELLO WORLD!
     [1 2 3 4 5 6 7 8 9 10 90 91 92]
     [1 2 3 4 5 6 7 8 9 10 90 91 92]


## Everything is an **Object** (even a class)

Time for some recursion: in Perl 6 everything is an object, including classes.
No wait, usually *an object is an instance of a class*, so how can a class be an object instance? Easy pal: **every type (class) exists at run-time also as an instance, and therefore can be assigned to variables**.
Quick, let's see a demonstration:

```perl
my $var;
$var = Str;
$var.perl.say;
```

Assigning the type `Str` to `$var` and printing it out produces the result `Str`, that is a string representation of the type name itself.
Let's see a more awkward example:

```perl
my $var;
for (1, Str, 2, Int) {
    $var = $_;
    $var.perl.say;
}
```

that produces as output

    1
    Str
    2
    Int

Therefore it is possible to assign to a variable either a value (e.g., `1`) or a type (e.g., `Int`).
The main difference is that **assigning a value makes the variable _defined_ (i.e., the `defined` method returns `True`), while assigning a type object will leave the variable _undefined_**.

### Forcing a type will not prevent for that *type class* to be assigned

Explicitly setting the type of a variable prevents, at run-time, a wrong assignment. Therefore, if the `$var` is declared as an `Int` it will accept only integer values:

```perl
my Int $var;  # explic Int type
for (1, Str, 2, Int) {
    $var = $_;
    $var.perl.say;
}
```

and the above piece of code will produce the following run-time error

    1
    Type check failed in assignment to $var; expected Int but got Str (Str)
      in block <unit> at test.p6 line 44

But it does not prevent the `$var` variable to be assigned the *type* itself, so the following will still work:

```perl
my Int $var;  # explic Int type
for (1, 2, Int) {
    $var = $_;
    $var.perl.say;
}
```

producing the following output

    1
    2
    Int

### Using the **:D** will prevent even the *type class* to be assigned

Perl 6 offers the `:D` adverb to place on the type of the variable to force the variable to be assigned to something **that is defined, that is a _value_**. Therefore, the following piece of code becomes broken:

```perl
my Int:D $var = 0;  # required an initial value!
for (1, 2, Int) {
    $var = $_;
    $var.perl.say;
}
```

and it breaks when the program tries to assign the `Int` object to the `$var`:

    1
    2
    Type check failed in assignment to $var; expected Int:D but got Int (Int)
      in block <unit> at test.p6 line 44

Therefore, **using `:D` the variable is forced to _defined_ values, i.e., values and not type objects, assignments**.


### Using the **:U** will prevent even the *values* to be assigned

In contrast to the `:D` adverb, Perl 6 provides the `:U` adverb that specifies that assignments can be done only to *undefined* values, that is type objects.
Therefore the following piece of code is broken:

```perl
my Int:U $var;
for (Int, 1, 2) {
    $var = $_;
    $var.perl.say;
}
```

this time the assignment of a value cannot be done:

    Int
    Type check failed in assignment to $var; expected Int:U but got Int (1)
      in block <unit> at test.p6 line 44


### Using the **:_** means both `:D` and `:U`

Using the `:_` adverb means explictly to let space for both a value and a type object, so the following code becomes working again:

```perl
my Int:_ $var; # same as Int $var;
for (Int, 1, 2) {
    $var = $_;
    $var.perl.say;
}
```
