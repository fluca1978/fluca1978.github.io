---
layout: post
title:  "plperl: invoking other subroutines"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- itpug
permalink: /:year/:month/:day/:title.html
---
`plperl` does not allow *direct sub invocation*, so the only way is to execute a query.


# `plperl`: invoking other subroutines

The official [plperl documentation](https://www.postgresql.org/docs/current/static/plperl-global.html) shows you a way to use a `subref` to invoke code shared across different `plperl` functions via the special global hash `%_SHARED`. While this is a good approach, it only works for code *attached to the hash*, that is a kind of closure (e.g., a dispatch table), and requires each time an initialization of the `%_SHARED` hash since `plperl` interpreters does not share nothing across sections.

The other way, always working, is to execute a query to perform the `SELECT` that will invoke the function.
As an example:

```sql
CREATE OR REPLACE FUNCTION plperl_trampoline( fun_name text )
RETURNS TEXT
AS $PERL$
   my ( $fun_name ) = @_;
   return undef if ( ! $fun_name );
   elog( DEBUG, "Calling [$fun_name]" );
   my $result_set = spi_exec_query( "SELECT $fun_name() AS result;" ); 
   return $result_set->{ rows }[ 0 ]->{ result };
$PERL$
LANGUAGE plperl;
```

so that you can simply do:

```sql
> select plperl_trampoline( 'now' );

      plperl_trampoline       
------------------------------
 2018-05-04 13:09:17.11772+02
```

The problem of this solution should be clear: it can work only for a set of functions with the same prototype.
In fact, while it could be simple to work around the argument passing (thank to some magic with Perl arrays), the return type and, most notably, its arity makes the approach not easily universal.

<br/>
<br/>
Another introspective approach could have been to use `pg_proc.prosrc` to translate the Perl code to an anonymous function on the fly, and put it into the `%_SHARED` global hash. However, this requires special care about arguments too, and makes it less than trivial to handle the function protytpe.


# An example of using `%_SHARED` to get sequence values

Once common issue when dealing with stored procedures is to get new values from sequences. While this is really trivial in `plpgsql`, and reduces to a single call to `nextval()`, it is not so simple in `plperl` where an `spi_exec_query()` has to be issued.
It is however possible to use the `%_SHARED` hash to add an handler for the same query:

```sql
CREATE OR REPLACE FUNCTION plperl_add_sequence_handler( s text )
RETURNS VOID
AS $PERL$

   my ( $sequence ) = @_;
   return 0 if ( ! $sequence );
   my $query = sprintf "SELECT nextval( '%s' )", $sequence;

   elog( DEBUG, "Query [$query]" );
   $_SHARED{ $sequence } = sub {
      return spi_exec_query( $query )->{ rows }[ 0 ]->{ nextval };
   };

$PERL$
LANGUAGE plperl;
```

The above function sets up an handler with the name of the sequence itself, and each time its code is executed a query is issued against the database to get the `nextval()`.
Therefore, it is quite simple to set-up a sequence value in a session and do a retrieval:

```sql
> SELECT plperl_add_sequence_handler( 'persona_pk_seq' );
-- and later

> DO language plperl $$
   elog( INFO, $_SHARED{ persona_pk_seq }->() );
$$;
```

In the above `plperl` code the coderef is invoked via a reference in the `%_SHARED` hash, in particular:

   $_SHARED{ persona_pk_seq }->()
   
so that is is easier to get a sequence value in a *Perl-way*.   
