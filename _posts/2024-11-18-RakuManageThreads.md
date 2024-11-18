---
layout: post
title:  "Deciding how many threads to use in a Raku application"
author: Luca Ferrari
tags:
- raku
permalink: /:year/:month/:day/:title.html
---
A few common tricks to manage the parallelism level in Raku.

# Deciding how many threads to use in a Raku application

During a presentation I gave about Raku, at the last Linux Day 2024 in Modena, an attendee asked me if there was a way to control how many parallel threads the virtual machine was using to run work.

Since I was running out of time, I gave the short answer as *"yes, you can configure at run-time the virtual machine"*. This is true, and in this post I explain how to achieve this.

First of all, assume we have the following piece of code that, in short, starts hundreds of code to be executed in parallel.

<br/>
<br/>
```raku
#!raku

my @promises;


say "Max threads set to %*ENV<RAKUDO_MAX_THREADS>" if %*ENV<RAKUDO_MAX_THREADS>;

for 1 .. 200 {
    @promises.push: start {
	sleep( 2 );
	"I'm thread $_ over $*THREAD".say;
	sleep( $_ );
	};
}


await @promises;

```
<br/>
<br/>


The code already reveals a trick to control the number of threads: the environment variable **`RAKUDO_MAX_THREADS`**.

Let's dig a little more on this.

## The `ThreadPoolScheduler`

Raku internally uses an instance of `ThreadPoolScheduler` to manage a pool of threads.
As you [can see from the documentation](https://docs.raku.org/type/ThreadPoolScheduler){:target="_blank"} the pool is initialized with zero initial threads and can host 64 threads at max.
 There is something more: you can specifiy also a `*` whatever number of threads as upper limit, indicating *as much threads as the operating system is allowing to me*, whatever "allowing" means (essentially, either a capability limit or the operating system surrendering).

 The scheduler can be initialized at boot, i.e., when the virtual machine starts, by means of the environmental variable **`RAKUDO_MAX_THREADS`**, that as you can imagine, controls the number of threads to set as the upper limit.

 However, being the `ThreadPoolScheduler` a class within the Raku object model, it is possible to programmatically use it:

 <br/>
 <br/>
 ```raku
#!raku

my @promises;


say "Max threads set to %*ENV<RAKUDO_MAX_THREADS>" if %*ENV<RAKUDO_MAX_THREADS>;

my $*SCHEDULER = ThreadPoolScheduler.new( max_threads => 16 );

for 1 .. 200 {
    @promises.push: start {
	sleep( 2 );
	"I'm thread $_ over $*THREAD".say;
	sleep( $_ );
	};
}


await @promises;

```
<br/>
<br/>


Assigning a specific scheduler to the `$*SCHEDULER` runtime variable forces the system to use such schedule, so the code using such schedule will be limited to the scheduler's number of threads.
Clearly, you can overwrite the environment variable by means of accessing the `$*SCHEDULER`:


<br/>
<br/>
```raku
#!raku

my @promises;


say "Max threads set to %*ENV<RAKUDO_MAX_THREADS>" if %*ENV<RAKUDO_MAX_THREADS>;

my $*SCHEDULER = ThreadPoolScheduler.new( max_threads => 16 );
say "Max threads for the default scheduler are: { $*SCHEDULER.max_threads }";

for 1 .. 200 {
    @promises.push: start {
	sleep( 2 );
	"I'm thread $_ over $*THREAD".say;
	sleep( $_ );
	};
}


await @promises;

```
<br/>
<br/>


and even if I set the environment variable, the new scheduler wins and takes control:


<br/>
<br/>
```shell
% export RAKUDO_MAX_THREADS=4

% raku test.p6
Max threads set to 4
Max threads for the default scheduler are: 16
...
```
<br/>
<br/>


It is also possible to change the scheduler setup while the parallel thing is running, even this can cause a lot of confusion:

<br/>
<br/>
```raku
#!raku

my @promises;


say "Max threads set to %*ENV<RAKUDO_MAX_THREADS>" if %*ENV<RAKUDO_MAX_THREADS>;

say "Max threads for the default scheduler are: { $*SCHEDULER.max_threads }";

for 1 .. 200 {

    @promises.push: start {
	sleep( 2 );
	"I'm thread $_ over $*THREAD with a max of { $*SCHEDULER.max_threads }".say;
	sleep( $_ );
    };
}

sleep( 20 );
$*SCHEDULER = ThreadPoolScheduler.new( max_threads => 16 );
await @promises;

```
<br/>
<br/>

The above program shows an output like the following:

<br/>
<br/>
```shell
% raku test.p6
Max threads set to 4
Max threads for the default scheduler are: 4
I'm thread 1 over Thread<4>(GeneralWorker) with a max of 4
I'm thread 2 over Thread<6>(GeneralWorker) with a max of 4
I'm thread 3 over Thread<7>(GeneralWorker) with a max of 4
I'm thread 4 over Thread<8>(GeneralWorker) with a max of 4
I'm thread 5 over Thread<4>(GeneralWorker) with a max of 4
I'm thread 6 over Thread<6>(GeneralWorker) with a max of 4
I'm thread 7 over Thread<7>(GeneralWorker) with a max of 4
I'm thread 8 over Thread<8>(GeneralWorker) with a max of 4
I'm thread 9 over Thread<4>(GeneralWorker) with a max of 4
I'm thread 10 over Thread<6>(GeneralWorker) with a max of 4
I'm thread 11 over Thread<7>(GeneralWorker) with a max of 4
I'm thread 12 over Thread<8>(GeneralWorker) with a max of 4
I'm thread 13 over Thread<4>(GeneralWorker) with a max of 16
I'm thread 14 over Thread<6>(GeneralWorker) with a max of 16
I'm thread 15 over Thread<7>(GeneralWorker) with a max of 16

```
<br/>
<br/>

Note how *nearly* 10 things run with a scheduler limit of `4` (10 = 2 seconds x 10 = 20 secs, which the `sleep` before changing the scheduler), then the new scheduler kicks in and the system starts using a `16` thread limit.
Clearly, it is not well defined the moment at which the system needs to cut the old scheduler and the new one, since the threads could have been already scheduled on the previous one, so use with very care.


# Conclusions

There are two main possibilities to change and control the amount of parallel threads a Raku application is going to use:
via the `RAKUDO_MAX_THREADS` environment variable, which by default is not set and leads to a max value of `64` threads, or by injecting a `ThreadPoolScheduler` in your application or context.
