---
layout: post
title:  "Perl 6 Grammars introduction"
author: Luca Ferrari
tags:
- perl6
permalink: /:year/:month/:day/:title.html
---

here there's a small introduction to Perl 6 new super-powerful feature **grammars** that allow you to *parse* pretty much everything in a smart and extensible way.

# An Introduction to Perl 6 `Grammar`s

A `[Grammar](https://docs.perl6.org/language/grammar_tutorial)` is a special *class* made almost entirely by *regular expressions*.
The idea is to define a set of methods, each one that returns a specific regular expression, and which composition provides a **parser** for a specific input domain.

In this sense, a Grammar is nothing more than syntactic sugar between regular expressions and classes. Being classes a grammar can be extended in the Object Oriented Way, and can have members in the same sense; being made by regular expressions, a grammar provides a few shortcut to define its workhorse by mean of such regexps.

## Basic rules for a grammar

The basic rules are somewhat simple:
1. the grammar is defined via the special keyword `grammar`;
2. each grammar must have a specific method named `TOP` which is the entry point for the parsing engine (let's say it is the equivalent of the method `MAIN` for a regular program);
3. methods that return a regular expression must be defined with either `token` or `rule`, with the latter implicitly adding white space handling within the regular expression;
4. regular expressions are defined within their methods, and therefore do not require any start-end sequence (e.g., `//`);
5. a regular expression method is accessed with the special syntax `<name>`
6. a grammar automatically provides the method `parse` which accepts the material to parse and, optionally, the `rule` to use for parsing (if none, `TOP` is used). Please note that `parse` automatically anchors the input begin and end.

# A simple example

Suppose there's a simple programming language that defines an assigment statement by means of `move`:

    move src to dst

where:
- `dst` can be a field, with a name made by letters and a number separated by a dot;
- `src` can be a field, a number, or a quoted string;
- commands and field are case insenstive, the command terminator is the end of line.

Therefore, what the grammar is supposed to parse, is something like the following:

```cobol
MOVE "HELLO" TO MASK.1
move MASK.1 TO mask.2
move 3.6 TO mask.1
```

## The Grammar

First of all try to start from the bottom: name every single part of the statement(s) and create a regular expression that can handle only such minimal part. Therefore, it starts simple as:

```perl
grammar MoveGrammar {
    token move      { :i move  }
    token separator { :i to }
    token TOP       { .* }
}
```

In the above example the `move` and `separator` methods provide regular expressions that can match, in a case insensitive way (`:i`), only the begin and middle of the command. The special method `TOP` is there just a placeholder to allow the grammar to be quickly tested: the compiler will emit an error if the method is not provided.

Now, with that simple grammar in place, it is possible to quickly test each simple *rule* by calling the `parse` method specifying the rule to use instead of `TOP`:

```perl
say MoveGrammar.parse( 'move', :rule<move> );
say MoveGrammar.parse( 'To', :rule<separator> );
```

that outputs

```shell
｢move｣
｢To｣
```

It is now time to improve the grammar with all the other little pieces, so that the final result appears as:

```perl
grammar MoveGrammar {
    token move   { :i move  }
    token separator { :i to }
    token field  { :i \w+\.\d+ }
    token number { \d+ [ \.\d+ ]? }
    token string { \" <[ \w | \s ]>+ \" }
    token src  {
            | <number>
            | <string>
            | <field>
    }
    token dst { <field> }

    token TOP { .* }
}
```

As readers can see, the `src` and `dst` methods have been implemented in order to respect the command layout `mote src to dst`. The `dst` is the simplest one, since it simply can be a `field` the same method is invoked. The `src` can be either a field, a number or a string, therefore the `src` method is a *or* selection amongst `number`, `string` or `field`.
The above can be quickly tested with something like the above:

```perl
say MoveGrammar.parse( 'move', :rule<move> );
say MoveGrammar.parse( 'To', :rule<separator> );
say MoveGrammar.parse( 'Mask.1', :rule<field> );
say MoveGrammar.parse( '22.1', :rule<number> );
say MoveGrammar.parse( '"Hello World"', :rule<string> );

say MoveGrammar.parse( '22.1', :rule<src> );
say MoveGrammar.parse( '"Hello World"', :rule<src> );
say MoveGrammar.parse( 'Mask.1', :rule<src> );

say MoveGrammar.parse( 'Mask.1', :rule<dst> );
```

that produces something like the following:

```shell
｢move｣
｢To｣
｢Mask.1｣
｢22.1｣
｢"Hello World"｣
｢22.1｣
 number => ｢22.1｣
｢"Hello World"｣
 string => ｢"Hello World"｣
｢Mask.1｣
 field => ｢Mask.1｣
```

## The rule to the `TOP`

Having all the small pieces in place, it is now time to assemble the definitive `TOP` method, this time using a `rule` and not a `token` since the command can have an arbitrary amount of whitespaces between the single pieces:

```perl
grammar MoveGrammar {
    token move   { :i move  }
    token separator { :i to }
    token field  { :i \w+\.\d+ }
    token number { \d+ [ \.\d+ ]? }
    token string { \" <[ \w | \s ]>+ \" }
    token src  {
            | <number>
            | <string>
            | <field>
    }
    token dst { <field> }

    rule TOP    { <move> <src> <separator> <dst> }
}

```

As readers can see, once the single pieces have been defined, composing `TOP` often reduces to repeat the language specification and results in a very readable piece of code.

Now, it is possible to parse several statements within a loop:

```perl
my @statements = qw/ MOVE "HELLO" TO MASK.1 /,
    qw/ move MASK.1 TO mask.2 /,
    qw/ move 3.6 TO mask.1 /;

for @statements {
    $parser.parse( $_ );
    say $/;
}
```

The method `parse` exploits `TOP` anchoring it to the very begin and end of the input string, in this case `$_`. The result is something like the following:

```shell
｢MOVE "HELLO" TO MASK.1｣
 move => ｢MOVE｣
 src => ｢"HELLO"｣
  string => ｢"HELLO"｣
 separator => ｢TO｣
 dst => ｢MASK.1｣
  field => ｢MASK.1｣

｢move MASK.1 TO mask.2｣
 move => ｢move｣
 src => ｢MASK.1｣
  field => ｢MASK.1｣
 separator => ｢TO｣
 dst => ｢mask.2｣
  field => ｢mask.2｣

｢move 3.6 TO mask.1｣
 move => ｢move｣
 src => ｢3.6｣
  number => ｢3.6｣
 separator => ｢TO｣
 dst => ｢mask.1｣
  field => ｢mask.1｣

```

As readers can see, the `dst` always resolves to a `field` match, and that is clearly the only one possibility.
On the other hand, the `src` could match against a `field`, a `number` or a `string`.


## Using actions

Being able to parse an input source is just the top level part, even if surely one of the more complex and useful one, that Perl 6 makes very simple and coherent.
But what if there's the need to *elaborate* more the values of single parse steps? For instance in order to see if the value is within a valid range, or how to warn the user if the same value gets assigned over and over or simply something else?

Perl 6 introduces the *action* objects: classes that can be passed as argument to the `parse` method of a grammar in order to manipulate further more the matching of a regular expression part. **Action objects are regular Perl 6 objects** but if they define a method with the same name of one in the grammar, accepting a `Match` object, it will be invoked alongside the matching grammar method.

Therefore, having defined the grammar above, it is possible to keep a count of which numbers and strings get assigned to fields with the following Perl 6 class:

```perl
class MoveGrammarAction {
    has @.numbers;
    has @.strings;

    method number( $/ ){ @!numbers.push: $/.Rat; }
    method string( $/ ){ @!strings.push: $/.Str; }
}
````


and usage with the above grammar results in passing it to the `parse` method:

```perl
my @statements = qw/ MOVE "HELLO" TO MASK.1 /,
    qw/ move MASK.1 TO mask.2 /,
    qw/ MOVE "World" TO MASK.1 /,
    qw/ moVE 50 TO MASK.1 /,
    qw/ move 3.6 TO mask.1 /;

my $parser = MoveGrammar.new;
my $handler = MoveGrammarAction.new;
for @statements {
    $parser.parse( $_ , actions => $handler );
    say $/;
}


$handler.numbers.say;
$handler.strings.say;
```

that produces the following final output, showing how it captured the numbers and strings assigned in the statements

    [50 3.6]
    ["HELLO" "World"]


## Mading by make: more than actions

While action objects are really interesting and appealing for evaluating and elaborating more the single match, Perl 6 offers another way to *attach* a piece of data to a match. The `make` and `made` methods on a `Match` object can respectively attach and retrieve a value.

As an example, suppose there is the need to count how many times a field is accessed.
The action class changes so that a new method `field` is added to be called back when the same `<field>` regular expression match from the grammar:

```perl
class MoveGrammarAction {
    # ... same as previous definition
    has %.field-counters;

    method field( $/ ) {
        %!field-counters{ $/.Str.uc }++;
        $/.make: %!field-counters{ $/.Str.uc };
    }
}
```

The `field` method does two things:
1. counts with an hash indexed by the name of the field the number of times the same field appeared;
2. it calls `make` on the match object to *notify* it about how many time the field has been seen so far.


It is now possible to call the `made` method on the match object to get back the evaluation from the action object:

```perl
for @statements {
    $parser.parse( $_ , actions => $handler );
    say "Field $/<dst><field> has been seen  { $/<dst><field>.made } times so far";
}
```

It is important to note that the match object does not store the `<field>` value at the top level, but under the `<dst>` level since the grammar dictates that `field` is a sublevel of `dst`.
The above does not keep care about the possible `<src><field>`, but if the same appears in both the source and destination it will be counted, so the only improvement to do is about printing out the source field, that can be easily done with a condition:

```perl
for @statements {
    $parser.parse( $_ , actions => $handler );
    say "Field $/<src><field> has been seen  { $/<src><field>.made } times so far"
           if ( $/<src><field> );
    say "Field $/<dst><field> has been seen  { $/<dst><field>.made } times so far"
          if ( $/<dst><field> );
}
```

The final output from the above code is the following:

    Field MASK.1 has been seen  1 times so far
    Field MASK.1 has been seen  2 times so far
    Field mask.2 has been seen  1 times so far
    Field MASK.1 has been seen  3 times so far
    Field MASK.1 has been seen  4 times so far
    Field mask.1 has been seen  5 times so far



## Getting rid of actions

If all is required to simply attach a value representation to the match object, it is possible to avoid the action objects at all and use *in-place makde*. Suppose for instance we need to attach interger values to numbers, therefore where a `<number>` is applied in the grammar and matches, it is possible to insert a call to `make`:

```perl
grammar MoveGrammar {
    # ... same as in previous definition
    token src  {
        | <number> { $/<number>.make: $<number>.Int }
        | <string>
        | <field>
    }

   # ... same as in previous definition
}
```

When the `<number>` branch is executed, the `make` method is called on the `<number>` match itself, so that the following piece of code will find out the values:


```perl
 for @statements {
     $parser.parse( $_ );
     $/<src><number>.made.say if ( $/<src><number> );

 }
```

resulting in printing out all and only the integer part of the number assignments.


Another interesting part is that `make` is also a *routine*: when called outside of an object context it default to the match `$/`. The cool stuff this means is that it is possible to short to the minimum when it is required to attach only the same value that matched:


```perl
grammar MoveGrammar {
    # ... same as in previous definition
    token src  {
        | <number> { make: $/<number>.Int }
        | <string>
        | <field>
    }

   # ... same as in previous definition
}
```

but this time the `make` will attach the value to the branch level the parser is in, that is the value can be retrieved by `$/<number>.makde`.


# Making an extensible grammar

When designing or refactoring a grammar it is possible to use the so called **proto** regular expressions, that allows for further *plug-in* of terms in a future extension of the grammar (e.g., a subclass of the grammar).

## Proto regexp to handle multiple branches

In the grammar defined in the previous section, the `src` method is composed by three branches: `number`, `string`, `field`. It is possible to transform `src` in a proto regexp so that a later branch can be added in future.

Rules to create the proto regexep are the following:
1. the main regexp, in the example `src` is defined with the special keyword `proto` and does not have any definition rather than `{*}`;
2. all branches must be separately defined in a method named after the proto regexp, witha `:sym` infix. Within such method the special regexp `<sym>` produces a match against the exact name of the regexp.

Therefore, the grammar becomes:

```perl
grammar MoveGrammar {
    token move   { :i move  }
    token separator { :i to }
    token field  { :i \w+\.\d+ }
    token number { \d+ [ \.\d+ ]? }
    token string { \" <[ \w | \s ]>+ \" }

    proto token src {*}
    token src:sym<number> { <number> }
    token src:sym<string> { <string> }
    token src:sym<field>  { <field>  }


    token dst { <field> }

    rule TOP    { <move> <src> <separator> <dst> }
}

```
If, in the future, a new possible `src` has to be plugged in, it does suffice to add it in a subclass. For instance, if there's the need for a rational number without the leading zero (e.g., `.7` for `0.7`). it could be done as follows:

```perl
grammar MoveGrammarX is MoveGrammar {
    token src:sym<nozerorational> { \.\d+ }
}
```

and substituting the grammar when the `parse` step happens provides the feature.

from the above example it should be clear that **proto regexp provide an extensible way to plug-in branches**.

With regard to `<sym>`, it is an handy shortcut for exact matches, so that for instance changing the grammar to the following:

```perl
grammar MoveGrammar {
    # ... same as previous definition
    proto token move {*}
    token move:sym<move> { <sym> }
    token move:sym<MOVE> { <sym> }

    # ... same as the above definition
}
```

The grammar will match only the strings `move` or `MOVE` (breaking the case insensitivity), so the above is the same as writing:

```perl
grammar MoveGrammar {
    # ... same as previous definition
    proto token move {*}
    token move:sym<move> { move }
    token move:sym<MOVE> { MOVE }

    # ... same as the above definition
}
```


but using `<sym>` provide a shorter way.
