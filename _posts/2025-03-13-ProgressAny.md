---
layout: post
title:  "Using Progress::Any::Output and ::TermMessage in the correct way"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
An annoying thing I did not get from `Progress::Any::Output`.

# Using Progress::Any::Output and ::TermMessage in the correct way

I use `Progress::Any` a lot in my command line applications in order to provide feedback.
Since when I switched to `Progress::Any::Output::TermMessage`, one of the most customizable (according to me) output provider, I found myself unable to configure it properly. In particular, the task name and title were not showing correctly even if I was setting the string format properly.

Most notably, even using different instances resulted in the progress monitoring reporting thw wrong target (i.e., units), no matter the effort I did to `reset` and set `target` and `pos`.

Finally, after a lot of trials, I found out that in order to have all the progress reporting to work properly, I had to **predeclare** the *tasks* when calling `Progress::Any::Output::set()`.

The following is an example of a working program:

<br/>
<br/>
```perl
use v5.40;


use Progress::Any;
use Progress::Any::Output;

my $batch_size = 20;

Progress::Any::Output->set( { task => 'ALFA' }, 'TermMessage', template => "[%n %t] [%P / %T] %m - %R", single_line_task => 1 );
Progress::Any::Output->set( { task => 'BETA' }, 'TermMessage', template => "[%n %t] [%P / %T] %m - %R", single_line_task => 1 );

my $progress = Progress::Any->get_indicator( task => "ALFA", title => 'an alfa effort', target => $batch_size );


$progress->start;


for ( 1 .. $batch_size ) {
    $progress->update( message => "alfa at $_" );
    sleep( 1 );
}


$progress->finish;




$batch_size /= 2;
$progress = Progress::Any->get_indicator( task => "BETA", title => 'a beta effort', target => $batch_size );


$progress->start;


for ( 1 .. $batch_size ) {
    $progress->update( message => "beta at $_" );
    sleep( 1 );
}


$progress->finish;

```
<br/>
<br/>


The important lines are at the beginning:

<br/>
<br/>
```perl
Progress::Any::Output->set( { task => 'ALFA' }, 'TermMessage', template => "[%n %t] [%P / %T] %m - %R", single_line_task => 1 );
Progress::Any::Output->set( { task => 'BETA' }, 'TermMessage', template => "[%n %t] [%P / %T] %m - %R", single_line_task => 1 );

```
<br/>
<br/>

the above creates the properties for the `ALFA` and `BETA` progress instances, so that later calls to `get_indicator()` return a fully functional progress reporter.

The problem with this approach, according to me, is that you need to remember to set the formatter for a specific task, otherwise you are getting a partially functional indicator even if you call `get_indicator()` with different task names.
