---
layout: post
title:  "Compute day working hours in PL/pgsql"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org


permalink: /:year/:month/:day/:title.html
---
How many working hours are there in a range of dates?

# Compute day working hours in PL/pgsql
A few days ago there was a very [nice thread in the `pgsql-general` mailing list](https://www.postgresql.org/message-id/20190827222741.GB19306%40panix.com) asking for ideas about how to compute working hours in a month.
<br/>
The idea is quite simple: you must extract the number of working days (let's say excluding sundays) and multiple each of them for the number of *hours per day* and then get the sum.
<br/>
**There are a lot of nice and almost *one-liner* solutions in the thread, so I strongly encourage you to read it all!**
<br/>
<br/>
I came up with my own solution, that is based on *functions*, and here I'm going to explain it hoping it can be useful (at least as a starting point). 
<br/>
<br/>
You can find the code, as usual, on my [GitHub repository related to PostgreSQL](https://github.com/fluca1978/fluca1978-pg-utils/blob/master/examples/functions/working_hours.sql).

## The workhorse function
One reason I decided to implement the alghoritm using a function was because I want it to be configurable. There are people, like me, that do a job where the working hours are different on a day-by-day basis. So, assuming the more general problem of computing the working hours between two dates, here there's a possible implementation:

```sql
CREATE OR REPLACE FUNCTION 
compute_working_hours( begin_day DATE,
                        end_day DATE,
                        _saturday boolean DEFAULT false,
                        _hour_template real[] DEFAULT ARRAY[ 8, 8, 8, 8, 8, 8, 8 ]::real[],
                        _exclude_days date[] DEFAULT NULL )
RETURNS real
AS $CODE$
DECLARE
  working_hours real := 0;
  working_days daterange;
  current_day date;
  current_day_hours real;
  skip boolean;
BEGIN
  -- check arguments
  IF begin_day IS NULL
     OR end_day IS NULL
     OR begin_day >= end_day THEN
     RAISE EXCEPTION 'Please check dates';
  END IF;


  IF _hour_template IS NULL THEN
     _hour_template := ARRAY[ 8, 8, 8, 8, 8, 8, 8 ]::real[];
  END IF;
  WHILE array_length( _hour_template, 1 ) < 7 LOOP
    _hour_template := array_append( _hour_template, 8 );
  END LOOP;

   -- create the working period date range
  working_days = daterange( begin_day, end_day, '[]');

  RAISE DEBUG 'Working days in the range %', working_days;

  current_day := lower( working_days );
  LOOP
     -- skip sundays
     skip := EXTRACT( dow FROM current_day ) = 0;
     -- skip saturdays if required
     skip := skip OR  ( NOT _saturday AND EXTRACT( dow FROM current_day ) = 6 );

     -- skip this particular day if specified
     skip := skip OR ( _exclude_days IS NOT NULL AND _exclude_days @> ARRAY[ current_day ] );

     IF NOT skip THEN
        current_day_hours := _hour_template[ EXTRACT( dow FROM current_day ) ];
     ELSE
        current_day_hours := 0;
     END IF;

     RAISE DEBUG 'Day % counting % working hours',
                 current_day,
                 current_day_hours;

     working_hours := working_hours + current_day_hours;
     current_day   := current_day + 1;
     EXIT WHEN NOT current_day <@ working_days;
  END LOOP;


  -- all done
  RETURN working_hours;


END
$CODE$
LANGUAGE plpgsql;
```

Let's consider the arguments: the first two are the dates you want to inspect, then there's a boolean that indicates if saturday is a working day or not. The `_hour_template` is a template with the amount of hours within each day (sunday first, which can be any value since sundays are never working days - at least I would it to be!). Last, an array of days to exclude from the computation (holidays, vacation, and so on).
<br/>
<br/>
The function computes a `working_days` date range including the begin and end date, and then uses a `current_day` single day date to iterate within the date range. In the main loop, there are checks to skip the current day in the case it is a sunday, or a saturday (and saturdays are not working days) or is included into the array of ecluded days.
<br/>
Then the tricky part: if the day has to be excluded, the working hours will be *zero*, otherwise the working hours will be extracted from the hour template. Working hours are then summed together.
<br/>
<br/>
Let's see this in action:
```sql
testdb=# select compute_working_hours( current_date, 
     current_date + 3, 
     false, NULL, ARRAY[ '2019-08-28' ]::date[] );
DEBUG:  Working days in the range [2019-08-28,2019-09-01)
DEBUG:  Day 2019-08-28 counting 0 working hours
DEBUG:  Day 2019-08-29 counting 8 working hours
DEBUG:  Day 2019-08-30 counting 8 working hours
DEBUG:  Day 2019-08-31 counting 0 working hours
compute_working_hours
-----------------------
                    16
```

*I wish not to work on my beautiful wife's birthday*, so within the three days I'm supposed to work only two and get `16` hours.
<br/>
<br/>
As you probably have noticed, the hour template is expressed as `real` values, so that it is possible to express even part of hours, like 8.5 to indicate 8 hours and half. Here probably the usage of `time` would have been a better choice, but with a little complication over the final sum, so I'm not yet convinced about providing such an implementation.


## Back to the real problem: computing within a month

Having the above function in place, it is now possible to *overload* it and provide a function that computes the working hours in a single month of the year:

```sql
CREATE OR REPLACE FUNCTION 
compute_working_hours( _year int,
                       _month int,
                       _saturday boolean DEFAULT false,
                       _hour_template real[] DEFAULT ARRAY[ 8, 8, 8, 8, 8, 8, 8 ]::real[],
                       _exclude_days int[] DEFAULT null )
RETURNS real
AS $CODE$
DECLARE
  _exclude_days_as_dates date[];
  current_index int;
BEGIN
  -- check arguments
  IF _year IS NULL THEN
     _year := extract( year FROM CURRENT_DATE );
  END IF;

  IF _month IS NULL THEN
     _month := extract( month FROM CURRENT_DATE );
  END IF;

  IF _exclude_days IS NOT NULL THEN
    FOR current_index IN 1 .. array_upper( _exclude_days, 1 ) LOOP
     _exclude_days_as_dates := array_append( _exclude_days_as_dates,
                                             make_date( _year, _month, _exclude_days[ current_index ] ) );
    END LOOP;
  END IF;

  RETURN compute_working_hours( make_date( _year, _month, 1),
                              ( make_date( _year, _month, 1) + '1 month - 1 day'::interval )::date,
                                _saturday,
                                _hour_template,
                                _exclude_days_as_dates );

END
$CODE$
LANGUAGE plpgsql;
```

As you can see, the function asks for the year and month (as well as other parameters like the hour template), computes the range of dates for the specified month and delegates to the former implementation the computation.
<br/>
<br/>
One part I'm not really proud of is the `_exclude_days` parameter, that in this version is an array of integers that I have to convert then in array of `date`s. On one hand, I wanted the function to have coherent parameters, so if I specify a single month and want to skip the day `28` I already know that's the `28`th day of that month, so it is just a noise to ask the user to input a `date`. On the other hand, the loop that converts `__exclude_days` into an array of dates named `_exclude_days_as_dates` is less than elegant!
<br/>
<br/>
By the way, how is this invoked?

```sql
testdb=# SELECT compute_working_hours( NULL, 
                                       NULL, 
                                       true, 
                                       NULL,  
                                       ARRAY[12, 15 ,29] );
DEBUG:  Working days in the range [2019-08-01,2019-09-01)
DEBUG:  Day 2019-08-01 counting 8 working hours
DEBUG:  Day 2019-08-02 counting 8 working hours
DEBUG:  Day 2019-08-03 counting 8 working hours
...
 compute_working_hours
-----------------------
                   192
```

And yes, I love *defaults* and so pretty much every parameter can be omitted at all and still get a pretty decent result.
