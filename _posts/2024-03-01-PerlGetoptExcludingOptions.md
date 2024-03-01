---
layout: post
title:  "Exclusive command line options in Perl applications"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
A simple way to exploit the option library to make your application to accept only one option at a time.

# Exclusive command line options in Perl applications

I almost always use [Getopt::Long::Descriptive](https://metacpan.org/pod/Getopt::Long::Descriptive){:target="_blank"} to parse the command line arguments in my scripts and applications.

One common use case I have is to use an option to instrument the mode the application is going to work, but ensuring that no more then one mode has been specified. For instance, imagine the application could have the following options (among the others):
- `--nuke` to destroy the database content;
- `--init` to initialize the database;
- `--backup` to backup the database content.

Clearly, it does not make any sense to specify more then one of the above options at a time.

But how to achieve this?

## A *manual* approach

A manual approach, is to count how many options have been specified on the command line and interrupt the application if more than one is found:

<br/>
<br/>
```perl
use Getopt::Long::Descriptive;
my ($opts, $usage) = describe_options(
				      'myapp.pl %o <some-arg>',
				      [ nuke => 'delete database content' ],
				      [ init => 'initialize the database' ],
				      [ backup => 'backup the database' ],
				     );


my $exclusive_options = 0;
for ( qw/ nuke init backup / ) {
    $exclusive_options++ if ( $opts->$_ );
}

say $usage and exit if ( $exclusive_options > 1 );

# ... application continues ...
```
<br/>
<br/>

The main idea is to handle into the `$exclusive_options` the number of options specified on the command line, exiting if the counting is greater than `1`.

The main disadvantage of this approach is that whenever the application gains a new option, both the `Getopt::Long::Descriptive` and the code block must be updated, so this approach is error prone.

## `Getopt::Long::Descriptive` approach

Luckily, `Getopt::Long::Descriptive` provides a way to bind an option to a set of enumerated values.
Moreover, since an option can be marked as `hidden`, it will not appear on the help screen, making this approach seamless.

<br/>
<br/>
```perl
use Getopt::Long::Descriptive;
my ($opts, $usage) = describe_options(
				      'myapp.pl %o <some-arg>',
				      [ do => hidden =>
  		 			    { one_of => [
						     [ nuke => 'delete database content' ],
						     [ init => 'initialize the database' ],
						     [ backup => 'backup the database' ],
						     ] } ],

				     );

# ... application continues here ...
```
<br/>
<br/>

The idea is to define a `do` option, that being `hidden` will not be shown on the `$usage` help screen.
The trick is to use **`one_of`** that accepts an array ref with all the options.

The final result will be that only the three `one_of` options will be specified on the `$usage` help screen, and setting more than option at the same time will result in the application stopping automatically:

<br/>
<br/>
```shell
% perl ~/tmp/test.pl --nuke --init
The 'nuke' parameter ("1") to "eval" did not pass the 'nuke implies do=nuke' callback: these options conflict; each wants to set the do: init nuke


myapp.pl [long options...] <some-arg>
        --nuke    delete database content
        --init    initialize the database
        --backup  backup the database

```
<br/>
<br/>

The error message reveals the trick: the `one_of` on `do` means that anyone of the option will set also the hidden `do` option, but only one will be authorized at a time.

In other words, passing for instance `--nuke` will result in `$opts->do` and `$otps->nuke` being set at the same time.
With this trick in mind, it is possible to enhance the application skeleton as follows:


<br/>
<br/>
```perl
use Getopt::Long::Descriptive;
my ($opts, $usage) = describe_options(
				      'myapp.pl %o <some-arg>',
				      [ do => hidden =>
  		 			    { one_of => [
						     [ nuke => 'delete database content' ],
						     [ init => 'initialize the database' ],
						     [ backup => 'backup the database' ],
						     ] } ],

				     );

say $usage and exit if ( ! $opts->do );   # do not know what to do!
# ... application continues here ...

```
<br/>
<br/>

I suggest you to pick a better name than `do`, since it is a Perl operator, but that works for me.
