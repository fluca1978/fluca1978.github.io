---
layout: post
title:  "DBIx::Class ResultSet::first method"
author: Luca Ferrari
tags:
- perl
- dbic
permalink: /:year/:month/:day/:title.html
---
A stupid misunderstanding of what `first` (and `last`) methods do

# DBIx::Class ResultSet::first method

A few days ago, while working on an application of mine, I spotted a bug where values that were supposed to be all different were effectively all the same.
At first, I thought my database was somehow corrupted, that is a fancy way of saying it was not in the state I was expecting, but the data was fine, so the problem should have been in the application.

Another, underestimated symptom of this behavior was that the application was too slow in fetching that piece of data.

After digging a little, I found a place where I was using the method `DBIx::Class::ResultSet::first` to get a single record out of a resultset. The problem was my misunderstanding of how this method work, hence this article to try to explain better how to bend `DBIx::Class` to what I want it to do.

Assume you have the following simple table (please note, this example is simple to keep the code short and to the point!):

<br/>
<br/>
```sql
testdb=> select * from person;
 pk |   name   | surname
----+----------+---------
  1 | Luca     | Ferrari
  2 | Diego    | Ferrari
  3 | Emanuela | Ferrari
(3 rows)

```
<br/>
<br/>

Assume you want to get out from the database the *first* record with a person name and surname, assuming the person could be duplicated (i.e., there could be more than one entry with the same name and surname).

This is how I did it, **and it is wrong**:

<br/>
<br/>
```perl
use Fluca::Schema;
use Fluca::Schema::Result::Person;


my $db = Fluca::Schema->connect( 'dbi:Pg:dbname=testdb;host=rachel;port=5432',
				 'luca',
				 'xxxxx' );

my $luca = $db->resultset( 'Person' )->first( { surname => 'Ferrari', name => 'Diego' } );
say "I found " . $luca->name;

```
<br/>
<br/>

In the above snippet of code, I was trying to get all the entries named `Diego Ferrari` and get the first one among all of them.
The problem is, the above code finds `Luca Ferrari`, so not what I want.


The fact is that `first`, as the documentation states, performs a `slice` (i.e., it calls `slice` method) on the result set.
**And most notably, `first` does not accept any argument!**

<br/>
<br/>
```
% perldoc DBIx::Class::ResultSet

...

first
    Arguments: none
    Return Value: $result | undef

    Resets the resultset (causing a fresh query to storage) and returns an
    object for the first result (or "undef" if the resultset is empty).

...
```
<br/>
<br/>

So my searching hash clauses where discarded at all, and the `first` was applied to **the whole result set**, i.e., all the records in the table.

Turning on the `DBIC_TRACE` environment variable shown that the executed query was `SELECT me.pk, me.name, me.surname FROM person me:`, and that sounded strange to me (more on this later on), but simply proves that `first` was not acting as I wanted.


So how to get the first record among a filtered result set? The solution is really simple: **chain `search` and `first` together**.

<br/>
<br/>
```perl
$luca = $db->resultset( 'Person' )
           ->search( { surname => 'Ferrari', name => 'Diego' } )
		   ->first;
```
<br/>
<br/>

And that's it!

The same considerations apply to `last`, because both methods are implemented by means of `slice`, and hence the same considerations apply to `slice` too.


## What is strange with the implementation of `first`?

The query about the first wrong usage of mine extracts all the result set:

<br/>
<br/>
```perl
my $luca = $db->resultset( 'Person' )
              ->first;
```
<br/>
<br/>

This results in the query `SELECT me.pk, me.name, me.surname FROM person me:` which is somehow strange to me. In fact, this means that the DBIx has to fetch all the result set and then discard all except the first tuple. It would have been better to implement the final query as `ELECT me.pk, me.name, me.surname FROM person me LIMIT 1;` since this can support also ordering, and instruments the database to avoid doing a full table scan.



# Conclusions

`DBIx::Class` is great, but sometimes it is not as such as intuitive to me. That's surely not a problem of `DBIx::Class` itself, rather a problem in me understanding how it works internally.

As a partial explanation, I have to say that switching between all this advanced ORMs (e.g., Pythons' QuerySet - yeah, I have to pay the rent) sometimes make me do things in ways that are fine for one of them but not for the other.
