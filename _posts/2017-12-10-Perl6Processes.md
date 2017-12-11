---
layout: post
title:  "Perl5 -> Perl 6: processes"
author: Luca Ferrari
tags:
- perl6
- perl5
- programming
permalink: /:year/:month/:day/:title.html
---
Perl 6 has a new infrastructure to handle processes and IPC.

# Perl5 -> Perl 6: prcoesses


## Running a single process

Running a single process in Perl 6 is as simple as invoking the `run` method. `run` is a method that builds up a new [`Proc`](https://docs.perl6.org/type/Proc) object that is responsible to encapsulate a running process.
In short, it is possible to state that `run` is the same as backticks to Perl 5, with the very exception that they return not the command output but a command handler (the `Proc` object).

In other words, to run a process in Perl 6 you need only:

```perl
my $command_to_execute = 'ps';
my $arguments = '-aef';
my $proc = run $command_to_execute, $arguments;
say 'Exit code ' ~ $proc.exitcode ~ ' exited with signal ' ~ $proc.signal;
say 'Command was ' ~ $proc.command;
$proc.out.say;

```

The process is immediatly started and executed, so the command `$command_to_execute` is launched with the `$arguments` parameters. Its output is printed to screen and captured into the `out` method.
The `exitcode` method provides the exit code of the command, and in the case the command has been killed via a signal, the `signal` provides the number of the signal (zero for a normal completion).

The `command` method provides an array of what was effectively run, in other words, the command line launched.

### Where is new? Use `run`, `spawn` and `shell`!

Why don't run a `Proc` via its `new` constructor?
First of all, the `new` constructor accepts only named parameters, and if you take a look at the documentation for [`Proc.new`](https://docs.perl6.org/type/Proc#method_new) you will see there's no way of specifying the command to run at all.

The command (and its arguments) are specified via `run` or `spawn` or `shell`, and these are the only entry points to run a process.
In other words:
- if you don't have a `Proc` instance, use **`run`**;
- if you have a `Proc` instance, use either **`spwan`** or `shell`.

For instance, using `spawn`:

```perl
my $proc = Proc.new;
$proc.spawn( $command_to_execute, $arguments );
```

The difference between `spawn` and `shell` is that the latter does not allow you to specify the arguments separately and runs the comamnd with the shell:

```perl
my $proc = Proc.new;
$proc.shell( $command_to_execute ~ ' ' ~ $arguments );
```



## Piping commands

Piping commands in Perl 6 is easier than in Perl 5, and you don't need to open file handles with the pipe symbol, but to simply *connect* two processes:

```perl
my $proc = run $command_to_execute, $arguments, :out;
my $grep = run 'grep', 'perl', :in( $proc.out );
```

The above code runs the first `$proc` command and passes it to `grep`. The `:out` and `:in` allows the connections between the processes.


# Asynchronous Processes

## Background: Promise and Supply

Before going into details about how to use async processes, let's introduce a couple of classes used to deal with async IPC: `[Promise](https://docs.perl6.org/type/Promise)` and `[Supply](https://docs.perl6.org/type/Supply)`.

The `Supply` is a thread-safe implementation of the *Observer* pattern, in other words **`Supply` does implement a thread-safe container for the input/output notifications. The idea is that `Supply` implements a _channel_** to which other instances can subscribe to receive the data stream. Suppliers can be of two main types:
- *live* they provide data only since you connect to them;
- *on demand* they provide the data from the beginning every time a new consumer attaches to it.

It is possible to create a `Supply` instance using the `tap` or `act` methods, to which you pass a block of code that will be executed once new data is available on the *channel*. The only difference between `tap` and `act` is that the latter executes the block of code within a single thread at a time, so it is a little more safe with respect to shared resources and critical sections.

A `Supply` can return also a `Promise`, a `Promise` is an instance of something that will become completed in the future, in other words **a `Promise` is an handler for a computation that is still going on and will complete in the next future**.

**A `Supply` provides new data to the subscribers via the `emit` method**.

It is important to note that a `Supply` is not instantiated via the `new` method, rather it is obtained by a `Supplier` object.
That said, the following piece of code provides a way to generate and consume events:

```perl
my $producer   = Supplier.new;
my $consumer_1 = $producer.Supply;

$consumer_1.tap( -> $event { say "Consumer 1: $event" } );
$producer.emit( 'HELLO WORLD!' );

my $consumer_2 = $producer.Supply;
$consumer_2.tap( -> $ev { say "Consumer 2: $ev" } );

$producer.emit( "Hello World $_!" ) for 1..3;
```

that produces the following output:

```shell
Consumer 1: HELLO WORLD!
Consumer 1: Hello World 1!
Consumer 2: Hello World 1!
Consumer 1: Hello World 2!
Consumer 2: Hello World 2!
Consumer 1: Hello World 3!
Consumer 2: Hello World 3!
```

As you can see, the `$consumer_2` is attached only to the second part of the stream, and therefore the very first `HELLO WOROLD!` is printed out by only the first consumer.


It is now time to get back to `Promises`s: a `Promise` is an handler for a computation that will end in the next future. A `Promise` can be in one of the following states:
- `Planned` if it is executing (or has been scheduled for execution);
- `Kept` in case of succesful completion;
- `Broken` in case of problematic completion.

A `Promise` can be created via the `new` method or via a few factory methods:
- `start` creates a `Promise` for the specified block of code with; the `Promise` will be kept on success of will be broken if an execption is thrown;
- `in` creates a `Promise` that will succeed (i.e., will be `kept`) after a specified number of seconds. This is usually used to create a timer or timeout promise;
- `at` similar to `in`, will provide a `Promise` that is kept at the specified time instant;
- `allof` provides a `Promise` that succeeds (i.e., it's kept) once all its inner promises have finished (either with success or abort);
- `anyof` provides a succesful `Promise` as soon as one of the inner promises has completed (wither succesfully or not).

A `Promise`, being asynchronously, could end after the end of the of the main program loop, therefore it is possible to force the caller to *wait* for ongoing computations. The `await` method is used to wait for a single `Promise` or an array of `Promises`s.

Once the promise is kept, the code provided by the `then` method is executed.

That being said, the following is a simple snippet of code that creates a computation that is performed 10 seconds after its creation:

```perl
say "Creating the promise: " ~ now;
my $do_lately = Promise.in( 10 );
$do_lately.then( { say "Promise elapsed! " ~ now } );
say "Promise launched!";
await $do_lately;
```

that produces:

```shell
Creating the promise: Instant:1513001393.523191
Promise launched!
Promise elapsed! Instant:1513001403.528402
```

Let's provide a slightly more complex example with multiple `Promise`s scheduled at different times:

```perl
say 'Creating the promises: ' ~ now;
my @promises;
for ( 1 .. 3 ) -> $_ {
     if ( $_ % 2 == 0 ) {
         my $p = Promise.in( $_ * 2 );
         $p.then( { say "Promise $_ completed! " ~ now } );
         @promises.push: $p;
     }
    else {
        my $p = Promise.at( now + $_ + 5 );
        $p.then( { say "Promise $_ completed! " ~ now } );
        @promises.push: $p;
     }
}

say 'End of Promises setup ' ~ now;
await @promises;
for @promises -> $p {
    say "OK ? $p.result() STATUS: $p.status()";
}
```

The `result` method returns `True` if the promise has completed with success, while the `status` method returns `Kept` or another promise status like `Broken` or `Planned`. Please note that calling `result` will wait on the `Promise`.

It is important to note that a `Promise` cannot be `Kept` or `Broken` once it has been `Planned`, in order to do such you need to pass thru a `Vow` object obtained via the `vow` method:

```perl
my $p = Promise.new;
my $vow = $p.vow;
$vow.break: 'Cancelled!';

@promises.push: $p;
```

that produces the following output:

```perl
An operation first awaited:
  in block <unit> at proc.p6 line 52

Died with the exception:
    Cancelled!
      in block <unit> at proc.p6 line 52

```

A `Promise` provides a `Supply` method to obtain the streams to which connect the streams of data.


## Putting it all together and running async processes

Having introduced `Supply` and `Promise` it is now possible to have a look at [`Proc::Async`](https://docs.perl6.org/type/Proc::Async) that holds an async process. Such class provides an interface similar (but separated) to the `Proc` one: it is possible to use the `Supply` for standard output and error, and thru `Promise` allows a manager control flow to instrument the execution flow.

```perl
my $do_later = Proc::Async.new( 'host', 'www.perl6.org' );
$do_later.stdout.tap( -> $event { "[STDOUT] $event".say } );
$do_later.stderr.tap(  -> $event { "[STDERR] $event".say } );


say "Launching ... ";
my $timeout = Promise.in( 10 );
await Promise.anyof( $timeout, $do_later.start );
say "Completed!";

```

The above code creates a `$do_later` process handler for resolving a host name. Another `Promise` is used as a ten second timeout, and then the master loop waits for the first one that completes via `Promise.anyof`. This simply means that either the host name completes or the timeout expires. Before that, the `stdout` and `stderr` streams of type `Supply` are attached to a very simple handler.

The above piece of code provides the following output:

```shell
Launching ...
[STDOUT] www.perl6.org has address 213.95.82.53

Completed!
```

## The `react` block

The above `tap`ping and `await`ing piece of code can be condensed within a so called **`react`** block, that is a kind og `given/when` designed for async operations.
The above piece of code becomes like the following:

```perl
my $do_later = Proc::Async.new( 'host', 'www.perl6.org' );
say "Launching ... ";

react {
    whenever $do_later.stdout {
        "[STDOUT] $_".say;
    }

    whenever $do_later.stderr {
        "[STDERR] $_".say;
    }

    whenever $do_later.start {
        say "FINISHED! " ~ $_.exitcode;
        done; # completes the react block
    }

    whenever Promise.in( 10 ) {
        "[TIMEOUT] expired!".say;
        done; #complete the react block
    }
}

say "Completed!";
```

The `react` block provides *automagic* tapping other `Supply` instances as well as *automagic* awaiting other `Promise` instances, resulting in a much more declarative piece of code.
More in detail **a `react` block runs all the `whenever` clauses and then `await`s for all of them to perform or for any of them to call `done`**.
Therefore when a `whenever` is performed against a `Supply` it does `tap` such `Supply`, when it is performed other a `Promise` it does `await` on it.
