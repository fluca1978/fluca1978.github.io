---
layout: post
title:  "Deleting all associations (bridge) with DBIx::Class (avoid delet_all!)"
author: Luca Ferrari
tags:
- perl
permalink: /:year/:month/:day/:title.html
---
How do you delete all the associations between two many-to-many ResultSet?

# Deleting all associations (bridge) with DBIx::Class (avoid delet_all!)

Shame on me!

I was kicked by this bug for weeks before realizing that I was using the great `DBIx::Class` in the wrong way.

I have two `ResultSet` classes: `Publication` and `Classifiction`, tied by a *many-to-many* association.

For some strange reason, sometime, I was loosing records on the `Classification` table, thus making all my tests to fail. It took me a lot to understand what was going on. I had the idea that, while doing a bulk loading of `Publication` something was going wrong, and in fact I was **deleting** all the associations with a `Classification` every time a publication was *re-loaded*. So chances were there was a `CASCADE` somewhere, but unluckily this was not the case:

<br/>
<br/>
```perl
package Fluca::Schema::Result::Publication;
# ...
__PACKAGE__->has_many(
  "j_publication_classifications",
  "Fluca::Schema::Result::JPublicationClassification",
  { "foreign.publication" => "self.pk" },
  { cascade_copy => 0, cascade_delete => 0 },
);
```
<br/>
<br/>

In order to catch the bug, once it was clear to me that the bulk loading was the responsible, or better, the moment at which classifications were disappearing, I decided to give a run using `DBIC_TRACE` environment variable. This is a special variable that, once set, makes `DBIx::Class` to output on standard error every SQL query it is executing.

The problem was, as you can imagine, that a bulk loading involves a lot of queries and a lot of objects, so it was not possible for a mere mortal to read the queries while scrolling on the screen. Quite easy, though, to redirect the standard error to a *huge* text log:

<br/>
<br/>
```shell
% perl my_bulk_loader.pl > dbic.log 2>&1
```
<br/>
<br/>

Then I searched for a `DELETE FROM j_publication_classification` and, with my great surprise, I found out a set of `DELETE FROM publication` queries.
Clearly, such queries were the responsible of loosing the `Classification` records in the database.
Therefore, I digged the code in order to see what was happening within the point were the `DELETE` were issued, and I found the following line:

<br/>
<br/>
```perl
$publication->classifications->delete_all;
```
<br/>
<br/>

**I was jumping, thru a `DBIx::Class::Relationship` to the `Classification` and deleting them all!**
That's why I was loosing records.

But why did I use `delet_all` to remove the associations at first? Because if you have a *has-many* relationship, that is what works. However, if the relationship is a *many-to-many*, thus using a bridge, the **correct way to delete only the associations (i.e., to some extent, the bridges) is to use `set_xxx` accessor**.

When you add a *many-to-many* association, `DBIx::Class` adds an `add_` and an `set_`  methods to work with related objects.

Therefore, the correct solution was:

<br/>
<br/>
```perl
$publication->set_associrations( [] );
```
<br/>
<br/>

Note that you need to pass an explicit empty list, or the method will complain if no argument is given. With this piece of code, the associations are deleted.


# Conclusions

`DBIx::Class` is a very great tool, but sometimes I need to start over with a clear mindset probably abused by years using other ORM technologies!
