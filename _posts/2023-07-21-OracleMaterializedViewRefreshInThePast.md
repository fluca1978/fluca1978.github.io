---
layout: post
title:  "Oracle ORA-12005: may not schedule automatic refresh for times in the past"
author: Luca Ferrari
tags:
- oracle
permalink: /:year/:month/:day/:title.html
---
A tricky error when dealing with materialized views and their scheduled refresh.

# Oracle ORA-12005: may not schedule automatic refresh for times in the past

Oracle provides materialized views that can be automatically refreshed. When declaring the materialized view, it is possible to specify:
- `START` the instant when the materialized view should be generated at first;
- `NEXT` the instant at which the next refresh of the view will happen.

<br/>
The `NEXT` part allows the specification of an instant in the future, even an expression, so that at the time the view is defined, the database schedules a new job into its internal scheduler to run later and perform the materialized view refresh.

<br/>
So far, so good!
<br/>
Except when it does not work!
<br/>
<br/>
Let's start easy, and define an expression to refresh the view lately this day, at `19` and `30`. This could be the `NEXT` clause:

<br/>
<br/>
```sql
NEXT trunc(sysdate) + 19/24 + 30/(24*60)
```
<br/>
<br/>

The idea is to:
- take into account the current date and time (`sysdate`)
- exclude the time part via `trunc(sysdate)`
- add `19` hours via `19/24`
- add `30` minutes via `30/(24*60)`.

Assuming that is in the future, the view will be created and a job will be scheduled to be run later at `19:30`.
<br/>
<br>/
When the view is going to be refrehed, the same `NEXT` clause will be evaluated again and the job will be rescheduled. This is the magic by which Oracle performs the materialized view update.
<br/>
<br/>
And is also the root of problems.
<br/>
**The job will fail!**
<br/>
Inspecting the job logs, reveals that the job has failed with the cause **`ORA-12005: may not schedule automatic refresh for times in the past`**.
<br/>
How can a future time be in the past?
<br/>
It is really simple, after all: *when the job executes it tryes to redefined (override) the view with a new `NEXT` statement that is computed to be at the very instant time (or a little in the past)*. At `19:30` the job will run, and will try to schedule the next refresh of the view for `19:30` of the very same date (because of `trunc(sysdate)`), and that will make the job to fail!
<br/>
in other words, even if it seems that the job is going to be scheduled in the future, **scheduling a job for the very same day is a bad idea** and will, sooner or later, re-evaluate with a *in-the-past* error.
<br/>
<br/>
The solution is quite simple, after all: add one day to re-schedule the task at the very same hour the next day. Therefore:

<br/>
<br/>
```sql
NEXT trunc(sysdate + 1) + 19/24 + 30/(24*60)
```
<br/>
<br/>

This time, when the job will run, it will reschedule itself for the same time but within the next day, and the expression will never be evaluated in the past.
