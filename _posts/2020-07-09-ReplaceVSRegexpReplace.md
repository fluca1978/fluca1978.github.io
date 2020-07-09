---
layout: post
title:  "replace vs regexp_replace"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org

permalink: /:year/:month/:day/:title.html
---
Some considerations about the usage of `replace` or `regexp_replace`.

# replace vs regexp_replace

While trying to help Stefan Stefanov with his `[pg_spreadsheetml](https://github.com/stefanov-sm/pg_spreadsheetml){:target="_blank"}` I came across something that would have been obvious, but not too much to me.
<br/>
The obvious thing is *`replace` is generally faster than `regexp_replace`*.
<br/>
The fact is that, probably due to my heavy usage of Perl and Raku, I tend to use regular expressions even where they are not really required, and that is why I tried to change a nested invocation of `replace` into one of `regexp_replace`. The [pull request, and in particular the commit](https://github.com/stefanov-sm/pg_spreadsheetml/pull/3/commits/0e931cc212572be9db190bb761ef7d758fd61b2e){:target="_blank"} did transform something like:

```sql
replace(replace(replace(s, '&', '&amp;'), '>', '&gt;'), '<', '&lt;');
```

into something like

```sql
regexp_replace( regexp_replace( regexp_replace( s, '&', '&amp;', 'g' )
                                , '>'
                                , '&gt;'
                                , 'g' )
                            , '<'
                            , '&lt;'
                            , 'g' );
```


<br/>
<br/>
Now, despite the newlines, the usage of `regexp_replace` resulted in slower code.
So we decided to benchmark, and I decided in particular to test it with `pgbench`.


## Testing with `pgbench`

I created [three sql scripts](https://github.com/fluca1978/fluca1978-pg-utils/tree/master/examples/regexp_replace_becnhmarking){:target="_blank"} that essentially do the following:
- loop from 1 to the `:scale`;
- build a single XML piece of code with a sligthly different content to avoid caching;
- perform the substitution in three different ways
  - with `replace`
  - with `regexp_replace`
  - with `regexp_replace` and backreferences
- store the results with timing (`clock_timestamp()`) into a table for later analysis.

<br/>
<br/>
I did run the tests in a way similar to the following:

```sh
% pgbench -s 300000 -f benchmark_regexp_replace_compact.sql -U luca testdb
```

<br/>
and at the end I asked to get the result for the type of test.

## Results

Getting the results is quite straightforward, and on my PostgreSQL 12.2 I got:

```sql
testdb=> SELECT replacement_type, avg( ms ), min( ms ), max( ms ) 
         FROM benchmark_replace GROUP BY replacement_type;
         
    replacement_type    |          avg           | min |   max    
------------------------|------------------------|-----|----------
 regexp_replace         | 2.0656612333436503e-05 |   0 | 0.039055
 regexp_replace_compact | 0.00018001079899881362 |   0 |  0.06716
 replace                |  4.885953333294914e-06 |   0 | 0.027875
(3 rows)
```


<br/>
that clearly show how `replace` is ten times faster than `regexp_replace` that in turns, is roughly ten time faster that a `regexp_replace` with backreferences, as you could expect (even if I was hoping for a lower difference due to a minor number of invocations of the function).
<br/>
It is also interesting that the maximum times pretty much are `200%` of the previous best case.

# Conclusions

Even if the presented approach cannot be considered a *good benchmarking*, it does emphasizes how it is important to use the simplest function available for the task, in this case `replace` when you don't need to do a regular expression magic.
