---
layout: post
title:  "Perl5 -> Perl 6: about the topic variable"
author: Luca Ferrari
tags:
- perl6
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
Perl 6 aggressively reduces the need for the topic variable. Or does not?

## Perl5 -> Perl 6: about the topic variable
-----

The topic variable ```$_``` is a well known variable that is used as a global variable when nothing is provided.
In Perl 5 the rule of thumb is that a lot of operators can modify/act the topic varaible when nothing more is provided.
As an example, consider the following piece of code:

```perl
my @bands = ( 'Foo Fighters', 'Soundgarden', 'Puddle of Mudd', 'Depeche Mode' );

for ( @bands ){
    say "I like $_ !";
}

my %albums_of = map { $_ => int( rand( 10 ) ) } @bands;

for ( keys %albums_of ){
    say "I've got $albums_of{ $_ } albums of $_ !";
}

```

that produces the following output:

```
I like Foo Fighters !
I like Soundgarden !
I like Puddle of Mudd !
I like Depeche Mode !
I've got 2 albums of Puddle of Mudd !
I've got 1 albums of Foo Fighters !
I've got 1 albums of Depeche Mode !
I've got 1 albums of Soundgarden !
```

As you can see, the *for* loop as well as *map* operates on ```$_``` (topic variable) by default.

Let's see this in Perl 6:

```perl
my @bands = 'Foo Fighters', 'Soundgarden', 'Puddle of Mudd', 'Depeche Mode';

for @bands { "I like $_".say; }

my %albums_of = @bands.map: { $_ => Int( 10.rand ); }
for %albums_of.keys { say "I've got %albums_of{ $_ } albums of $_ !"; }
```

Nothing really impressive here: operators we were used to work with the topic variable remain pretty much the same.
What is new in Perl 6 is that **the [topic variable can be omitted](https://docs.perl6.org/syntax/$_), so that you can find method calls on no-object at all**, imagine the
topic variable is there. Let's see this in action:

```perl
for @bands { .say; }

# it is the same as
# for @bands { $_.say; }
```

As another example consider the following:

```perl
my $when = Date.today( formatter => { "%02d / %02d / %04d".sprintf: .day, .month, .year ; } );

$when.say;
```

where within the block for the formatter the default object refence is ```$_``` since none has been specified as prototype.
You can name the topic variable as you want by either place it explicitly or use a named variable (signature):

```perl
my $when = Date.today( formatter => -> $o { "%02d / %02d / %04d".sprintf: $o.day, $o.month, $o.year ; } );
```
