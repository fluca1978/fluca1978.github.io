---
layout: post
title:  "Estimating row count from explain output...in Perl!"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perl
permalink: /:year/:month/:day/:title.html
---
After having read the interesting post by [Laurenz Albe](https://www.cybertec-postgresql.com/en/count-made-fast/#comment-4409216441) on how to use `EXPLAIN` to get a quick estimate of a query count, I decided to implement the same feature in Perl.

# Estimating row count from explain output...in Perl!

At the end of his blog post, [Laurenz Albe](https://www.cybertec-postgresql.com/en/count-made-fast/#comment-4409216441) shows how to use a *quick and dirty* function to estimate the number of rows returned by an arbitrary query.
<br/>
<br/>
While I don't believe it is often a good idea to judge the size of a query by the optimizer guesses, the approach is interesting. Laurenz shows how to exploit the `JSON` format and query facilities to extract data from the `EXPLAIN` output, why not using Perl to crunch the textual data?

So [here it is](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/functions/explain_row_estimate.plperl.sql) a simple implementation to extract the estimate within Perl:

```perl
CREATE OR REPLACE FUNCTION plperl_row_estimate( query text )
RETURNS BIGINT
AS $PERL$

   my ( $query ) = @_;
   return 0 if ( ! $query );
   $query = sprintf "EXPLAIN (FORMAT YAML) %s", $query;

   elog( DEBUG, "Estimating from [$query]" );
   my @estimated_rows = map { s/Plan Rows:\s+(\d+)$/$1/; $_ }
                        grep { $_ =~ /Plan Rows:/ }
                        split( "\n", spi_exec_query( $query )->{ rows }[ 0 ]->{ "QUERY PLAN" } );

   return 0 if ( ! @estimated_rows );
   return $estimated_rows[ 0 ];
$PERL$
LANGUAGE plperl;
```

Let's see an example in action:

```sql
testdb=> select plperl_row_estimate( 'SELECT p.* FROM persona p JOIN persona k on k.pk = p.pk WHERE k.eta = 40' );

 plperl_row_estimate 
---------------------
               69500
```

How does the function works? The main trick is at this point in code:

```perl
   my @estimated_rows = map { s/Plan Rows:\s+(\d+)$/$1/; $_ }
                        grep { $_ =~ /Plan Rows:/ }
                        split( "\n", spi_exec_query( $query )->{ rows }[ 0 ]->{ "QUERY PLAN" } );

```

where thru `spi_exec_query` an `EXPLAIN` is executed and its format, in `YAML` is split into an array of strings, one entry per line. Such array, is then passed to `grep` to exclude all rows that do not contain information about the row estimation. Last, `map` extracts the numeric value from such lines. 
<br/>
<br/>
After that, therefore, there is an array of `@estimated_rows` entries where each one contains the rows estimatation of each plan node, with the outer node in the begin of the array. Such single position is therefore returned by the function and all the others are dropped away.
<br/>
<br/>
As a final note, please consider that such function accepts an arbitrary piece of text and tries to execute it as a query, *therefore it must be used carefully to avoid SQL-injection and problems alike*.
