---
layout: post
title:  "ORA-01036: illegal variable name/number (DBD ERROR: OCIBindByName)"
author: Luca Ferrari
tags:
- perl
- oracle
permalink: /:year/:month/:day/:title.html
---
How far Oracle errors can lead you!

# ORA-01036: illegal variable name/number (DBD ERROR: OCIBindByName)

While I was developing a *simple* **DBI** Perl application that was going to insert, thru prepared statements, stuff into an Oracle database, I encountered this error: `ORA-01036: illegal variable name/number (DBD ERROR: OCIBindByName)`.
<br/>
What the hell? I was not using named parameters, but positional ones, and I was not having the colon `:` in my query statement.
<br/>
The error showed up once I added a few columns to my `INSERT` statement, but I was unable to spot it at glance, and no, there was no *useless* `<*>` simbol in the Oracle error message.
<br/>
The code was something like the following:

<br/>
<br/>
```perl
my $sql =<<"SQL";
INSERT INTO foo( a, b, c )
VALUES (
       ?,
	   ?,
	   ?
	   );
SQL

$statement = $database->prepare( $sql );

# ...
my @values = ( $item->{ a }, 'foo', $stuff );
$statement->execute( @values ) or die "Damn!";

```
<br/>
<br/>

The above is fine, and it works, of course substituting the table and column names with something that does make sense.
However, when I added a couple of columns, the error prevented my script to work:


<br/>
<br/>
```perl
my $sql =<<"SQL";
INSERT INTO foo( a, b, c, d , e )
VALUES (
       ?,
	   ?,
	   ?
	   ?, ?
	   );
SQL

$statement = $database->prepare( $sql );

# ...
my @values = ( $item->{ a }, 'foo', $stuff, $data->{ d }, $prices->{ e } );
$statement->execute( @values ) or die "Damn!";

```
<br/>
<br/>

Can you spot the problem?
<br/>
**I forgot to add a comma `,` before the first added question mark for the appended new columns!**
<br/>
This is simply a **syntactic error**, nothing semantic that can deal with binding! Why is it Oracle throwing such a misleading error?
<br/>
Anyway, double checking the list of question marks and commas fixed the problem.


## Oracle is really misleading you

Yes, I think so!
<br/>
While in the above description the error, related to a missing comma, was referring to a binding problem, if you remove a comma from the list of column names, the error changes to `ORA-00917: missing comma (DBD ERROR: error possibly near <*> indicator`.
<br/>
Really Oracle, are you kidding? You can spot a missing comma in the column list but not in the value list?
<br/>
Now, someone could argue that it is simpler to syntax check the column names list rather than the values, but at least provide an hint explaining that *it could be, the user has forgotten a comma*!
