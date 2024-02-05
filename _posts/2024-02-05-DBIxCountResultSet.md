---
layout: post
title:  "DBIx::Class Tricks: the not-so-scalar ResultSet"
author: Luca Ferrari
tags:
- perl
- dbiclass
permalink: /:year/:month/:day/:title.html
---
A trick when counting tuples using DBIx::Class.

# DBIx::Class Tricks: the not-so-scalar ResultSet

I tend to write bulk loaders that work in batches: I gave the application an amount of work to do, and then expect that the application does exactly that amount of work an no more.

Except when the application decides to do more work!

The scenario is like the following one:
- there is a table where every entry has a `valid` field, that if false means that the entry has to be *worked out* (whatever it means to you);
- for every entry worked out, decrease the original batch size by one unit and continue;
- exit or stop when the batch size hits zero.


Pretty simple, uh? What can go wrong with this workflow?

A wrong `DBIx::Class` query can ruin the day!

<br/>
<br/>
```perl
my $count_entries_to_do = $db->resultset( 'Entry' )->search( { valid => 0 } );
if ( $count_entries_to_do > 0 ) {
   # do stuff on all the entries
   # ...
   # and remove them from the batch size
   $batch_size -= $count_entriess_to_do;   # argh !
}
```
<br/>
<br/>

Can you see the problem with the above piece of code?

Well, `search` returns a `DBIx::Class::Resultset`, which **in scalar context evaluates to the tuple count**, *but this does not mean it is a scalar value*!
So, whenever you evaluate the scalar variable associated to the `DBIx::Class::ResultSet` object, `DBIx::Class` will perform a new query, so changing the count.
Imagine you have `20` entries, and the batch size is set to `20`: you are expecting that at the end the expression `$batch_size -= $count_entries_to_do` will evaluate to `0`, but in reality the expression will be evaluated to `20` because the query will run again, finding no `valid => 0` entries.

This is not immediatly understandable, because if you set the batch size to `10` and the total number of entries to be processed is `20`, you will end up with the expected result of `10` remaining, but next runs will raise the problem.

Clearly, this is a **super power feature of `DBIx::Class`**, but it does not work out well in this scenario!

The solution, obviously, is to say to `DBIx::Class` that we need effectively the counting, so a scalar value, not an object that can evaluate to a scalar:

<br/>
<br/>
```perl
my $count_entries_to_do = $db->resultset( 'Entry' )->count( { valid => 0 } );
if ( $count_entries_to_do > 0 ) {
   # do stuff on all the entries
   # ...
   # and remove them from the batch size
   $batch_size -= $count_entriess_to_do;   # ok!
}

```
<br/>
<br/>
