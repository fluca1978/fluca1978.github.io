---
layout: post
title:  "Perl5 IO::Tee and verbose"
author: Luca Ferrari
tags:
- perl
- programming
permalink: /:year/:month/:day/:title.html
---
I tend to use the *verbose* option on all my applications, but how to deal with logging? Easy, with `IO::Tee` and a little trick.

# Perl5 IO::Tee and verbose
-----

Every stand-alone application I write do have a `Getopt` cycle, often `Getopt::Long::Descriptive`, and includes a **verbose** option to really see what is going on while the application is running.

Now, handling the verbose option is easy and not invasive at all thanks to the well known postfix conditionals:

```perl
say 'Here doing stuff...' if ( $opts->verbose );
```

But how to print and log at the same time? Well, it should be easy thanks to `IO::Tee` (which is an implementation similar to `tee(1)`).
But wait a minute, something like that will not work as I would like:

```perl
my $tee = IO::Tee->new( \*STDOUT, IO::File->new( 'log.txt', 'w' ) );
...
say {$tee} 'here doing some stuff' if ( $opts->verbose );
```

Why is it not working as expected (or at least as I want)? **Because it is printing the message to both `STDOUT` and the log *only* if the verbose mode is on!**
But the idea behind a log is to be annoying with verbose messages while the user is not annoyed with other messages on the screen, so **I want messages to be placed on the log *always* without any regard to the verbosity of the program run**.

So how to achieve that? The quick and dirty solution is to replicate any message to the respective file handle, so throwing away the `IO::Tee` at all:

```perl
say 'working..' if ( $opts->verbose );
say {$log} 'working..';
```

Of course it is nothing scalable and maintanable.
So let's pick up again the `IO::Tee` and apply the following idiom:

```perl
my $output = IO::File->new( $log_file_name, 'w' );

...
# options check
if ( $opts->verbose ){
    $output = IO::Tee->new( \*STDOUT, $output );
}

...
# and then
say ${output} 'working';
```

So what happened?
First of all I define a `$output` file handle to the log file. In this way, **each time I print to the `{$output}` I'm sure the message hits at leas the log file**.
Later, when I can check the options the user has entered for this particular run, **I overwrite the `{$output}` file handle with an `IO::Tee` (that exploits the `$output` itself) with the `STDOUT`**. This way, each time I `say` something, I'm sure it will be at least recorded in the logs, and moreover it will not be subjected to a postfix conditional.

As trivial as it can be, I use this approach quite often.
**Of course this does not substitute a more complex and feature rich *logging* mechanism**, rather it proposes a quick idiom to handle message to a log (always) and the standard output (if verbosity mode is on).


As a result of this post, I decided to try to manipulate the `IO::Tee` module to place an `add` method in order to allow for someone to add an handle after the object has been created. Please see the [Pull Request #2 here](https://github.com/neilb/IO-Tee/pull/2).
