---
layout: post
title:  "pgagroal log rotation and formatting"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
My small contributions to `pgagroal`.

# `pgagroal` log rotation and formatting

A few weeks ago I implemented a small contribution to `[pgagroal](https://agroal.github.io/pgagroal/){:target="_blank"}`, the *high-performance* PostgreSQL connection pooler, in order to implement *log rotation* and *log formatting*.
<br/>
At last, my contribution was [accepted and merged](https://github.com/agroal/pgagroal/pull/216){:target="_blank"}, but I did not get enough time to write on this until now.

## The issues

As you can read in the issues about [log rotation](https://github.com/agroal/pgagroal/issues/45){:target="_blank} and [log formatting](https://github.com/agroal/pgagroal/issues/44){:target="_blank"}, `pgagroal` was born with a very minimal support for logging.
With "minimal" I mean that the log file was not able to be `strftime(3)` compatible, therefore no placeholder were available in the log filename, and at the same time there was no rotation of logs at all.
<br/>
Therefore I decided to try to implement both these features, and here there is a short description of what I did.

### `strftime`

This has been the first place to start: allow the support of `strftime(3)` compatible strings in the `log_filename` parameter.
This has been quite easy, since the only need was to use `strftime` when [opening the log file](https://github.com/agroal/pgagroal/commit/ec08c52cc3a1589fa413e6af2141d01b5f9fa32a#diff-832ef7b42a93f7c071fa992df0be71e172ab68c7c85d8e87c9297c0225ddfbadR243){:target="_blank"}.

### Log rotation

This was much bigger to implement. First of all, I have to ensure that every time a new rotation was required, the system was able to rotate the logs.
<br/>
I decided to implement the *check* once a new log entry was outputted. This is clearly an unefficent approach, since the system is going to check the rotation needs much more than it is required, and can also slow down the logging system. However, the idea is that `pgagroal` is not going to log so much data to be impacted by the continuos check for rotation. Moreover, this was a kind of forced choice, since the logging is not done via a separated process.
<br/>
The next step was to ensure that, once a log rotation is required, the system can rotate the log file. In order to do tat two changes were required:
- the `log_filename` should be able to support different names, in particular becoming a `strftime(3)` compatible strings;
- the log file cannot be opened only in the application startup, but I needed a dedicated function to re-open the log file with a new name if needed.
<br/>
In order to speed up a little the activities, I also provided an utility function to test if log rotation was enabled. Therefore, in the case the rotation was disabled, nothing of the above will ever happen.
<br/>
Of course, the user must have some parameters to control log rotation, so I introduced `log_rotation_age` and `log_rotation_size`, both accepting strings that have to be converted respectively in seconds and bytes. And I provided the parsing functions as well.
<br/>
Rotating the log on the size basis was quite simple: I have to test if the file size has exceeded the `log_rotation_size`. It was much more difficult to implement the rotation by age.
<br/>
The rotation by age was implemented like this: a global variable keeps track of the last time the log file has been opened or reopened. Then, when I need to check for log rotation, I count the current time and get the difference between the current time and the last time the log file has been opened, if such number of seconds is greater than  `log_rotation_age`, the log must be closed and re-opened.
<br/>
<br/>
There is an important consequence in the above logic: the rotation does not happen **exactly** when it is supposed to happen, but always with a size/time delay. In other words, it is allowed for a log file to exceed by a single log entry (therefore, a feew bytes) the `log_rotation_size`, as well as not time based rotation will happen before a new log entry is flushed. This means that on a low busy server, you could see rotation to happen much later than what you configured.

<br/>
Last, there was to implement a some kind of *truncation* when a log rotates. Luckily, there was already such parameter, named `log_mode`, for opening a log file in append or truncate mode. I reused such logic in the re-opening log file function.

### Log Formatting

The last piece to add to the picture was the log formatting option. This was, after all, quite easy: every log entry was flushed with a `strftime(3)` fixed preamble; it was sufficient to provide an option `log_line_prefix` to use as a variable preamble to `strfime(3)`.


### Glance at contributed code

Here you can find a glance at the [contributed code](https://github.com/agroal/pgagroal/commit/ec08c52cc3a1589fa413e6af2141d01b5f9fa32a#diff-832ef7b42a93f7c071fa992df0be71e172ab68c7c85d8e87c9297c0225ddfbadR243){:target="_blank"}:
- `log_rotation_enabled()` returns *true* if the log rotation is active. The log rotation is automatically disabled if some configuration parameter is not set accordingly;
- `log_rotation_disable()` turns off the log rotation. This is done as a last resort when the configuration is miswritten;
- `log_rotation_required()` checks if **now** is required a log rotation, either by age or size;
- `log_rotation_set_next_rotation()` computes the next *age* at which a rotation by time should be triggered;
- `log_file_open()` opens or re-opens the log file using `strftime(3)` agains the `log_filename` configuration parameter. Every time the rotation must happen this function is invoked;
- `log_file_rotate()` performs the log rotation.

<br/>
As an example, the `log_file_rotate()` function is really simple:

<br/>
<br/>

``` c
void
log_file_rotate(void)
{
   if (log_rotation_enabled())
   {
      fflush(log_file);
      fclose(log_file);
      log_file_open();
   }
}
```
<br/>
<br/>

and as you can see, it flushes the current log and re-opens it.
And all the magic happens when the log entry is spurted:

<br/>
<br/>

``` c
 vfprintf(log_file, fmt, vl);
            fprintf(log_file, "\n");
            fflush(log_file);

            if (log_rotation_required())
            {
               log_file_rotate();
            }
```
<br/>
<br/>

Briefly, after the `fprintf` and the `fflush` the system asks itself if a log rotation is required, and in case, rotates the log file.


# Conclusions

While `pgagroal` has a basic logging mechanism, this contribution provides the log rotation features in a semi-precise way.
Contributing to this was fun, even if hard in some aspects, in particular because it's way too long since I develop something in C.
<br/>
`pgagroal` is a promising project, and I'm sure it is going to quickly show all its potential!
