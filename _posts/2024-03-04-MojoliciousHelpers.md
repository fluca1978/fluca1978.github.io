---
layout: post
title:  "Mojolicious Helper Methods"
author: Luca Ferrari
tags:
- mojolicious
- perl
permalink: /:year/:month/:day/:title.html
---
What are helper methods?

# Mojolicious Helper Methods

Mojolicious helper methods are a very nice feature that allows the **injection** of a method into controllers.
Therefore, such methods will be *shared* among all the controllers (and the application itself) and can be used in a way like `static` methods in other languages.

As an example, in order to share the same way to connect to the database (using `DBIx::Schema`), it is possible to implement the following helper:

<br/>
<br/>
```perl
sub startup {
   # ...

	# helper to find the db_schema
    $self->helper( db_schema => sub {
		       my ( $self ) = @_;
		       return MyDBIXSchema::Schema->connect( 'dbi:Pg:xxxx' );
		   } );

}
```
<br/>
<br/>

Once the helper method is installed, it can be called directly from a controller by means of `$self->db_schema`.

This is a really powerl yet simple way to define shared code without having to reference explicitly modules and subroutines in other pieces of code.
Moreover, being shared code, this helps in scalability, since changing the application configuration means changing the helper implementation everywhere.
