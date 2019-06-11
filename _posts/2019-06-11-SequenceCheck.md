---
layout: post
title:  "Checking the sequences status on a single pass"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
It is quite simple to wrap a couple of queries in a function to have a glance at all the sequences and their cycling status.

# Checking the sequences status on a single pass
The catalog `pg_sequence` keeps track about the definition of a single sequence, including the increment value and boundaries. Combined with `pg_class` and a few other functions it is possible to create a very simple administrative function to keep track about the overall sequences status.
<br/>
<br/>
I've created a `seq_check()` function that provides an output as follows:
```sql
testdb=# select * from seq_check() ORDER BY remaining;     
        seq_name        | current_value |    lim     | remaining  
------------------------|---------------|------------|------------
 public.persona_pk_seq  |       5000000 | 2147483647 |     214248
 public.root_pk_seq     |         50000 | 2147483647 | 2147433647
 public.students_pk_seq |             7 | 2147483647 | 2147483640
(3 rows)
```

As you can see, the function provides the current value of the sequence, the maximum value (limit) and how much values the sequence can still provide before it overflows or cycles. For example, `persona_pk_seq` has remained with 214248 values to provide. Combined with the current value, that is 5000000, this provides hint about the fact that the sequence has probably a too large increment interval.
<br/>
<br/>
The code of the function is as follows:

```sql
CREATE OR REPLACE FUNCTION seq_check()
RETURNS TABLE( seq_name text, current_value bigint, lim bigint, remaining bigint )
AS $CODE$
DECLARE
  query text;
  schemaz name;
  seqz    name;
  seqid   oid;
BEGIN

  FOR schemaz, seqz, seqid IN   SELECT n.nspname, c.relname, c.oid
                         FROM   pg_class c
                         JOIN   pg_namespace n ON n.oid = c.relnamespace
                         WHERE  c.relkind = 'S' --sequence
                    LOOP

     RAISE DEBUG 'Inspecting %.%', schemaz, seqz;

     query := format( 'SELECT ''%s.%s'', last_value, s.seqmax AS lim, (s.seqmax - last_value) / s.seqincrement AS remaining  FROM %I.%I, pg_sequence s WHERE s.seqrelid = %s',
                      quote_ident( schemaz ),
                      quote_ident( seqz ),
                      schemaz,
                      seqz,
                      seqid );

     RAISE DEBUG 'Query [%]', query;
     RETURN QUERY EXECUTE query;
  END LOOP;


END
$CODE$
LANGUAGE plpgsql
STRICT;
```

As you can see, the main query is a join between `pg_sequence` and data extracted directly from `pg_class`. The function iterates on all sequences within the system, and this means *the function must run with administrator privileges*.
<br/>
<br/>
I use this handy function to check the status on other machines, and quite frankly I've not yet come to `remaining` being near to zero, therefore I can sleep well at night:

```sql
=# select * from seq_check() order by remaining;
         seq_name          | current_value |         lim         |      remaining      
---------------------------|---------------|---------------------|---------------------
 t.root_pk_seq             |        201338 |          2147483647 |          2147282309
 respi.rosseni_tmp_pk_seq  |         16673 |          2147483647 |          2147466974
 respi.pull_status_pk_seq  |         14603 |          2147483647 |          2147469044
 respi.tipo_rossene_pk_seq |             8 |          2147483647 |          2147483639
 respi.root_pk_seq         |     140509487 | 9223372036854775807 | 9223372036714266320
 cron.jobid_seq            |             1 | 9223372036854775807 | 9223372036854775806
```

Of course, it is quite easy to improve the function adding, for instance, a percent ratio or a *near-to-cycle* flag.
