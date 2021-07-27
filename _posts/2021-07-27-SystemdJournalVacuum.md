---
layout: post
title:  "systemd: cleaning the space used by logs (journal)" 
author: Luca Ferrari
tags:
- linux
- systemd
permalink: /:year/:month/:day/:title.html
---

How to reduce the size of the logs used by `systemd`.

# systemd: cleaning the space used by logs (journal)

`systemd` has its own logging system, that is managed by `journalctl`.
<br/>
The idea is straightforward: instead of doing textual logging, `systemd` uses its own format for logging and `journalctl` is the way to access, read and manage the logs.
<br/>
Usually the logs are stored under `/var/log/journal`, and this can grow to a very large amount of space.
<br/>
`journalctl` provides special commands to clean up old logs: the commands are named `vacuum` and can be any combination of
- `vacuum-size` indicates the max amount of data to keep in the logs;
- `vacuum-time` indicates the max window of time to keep in the logs (i.e., logs within the last two days);
- `vacuum-files` indicates how many file of logs to keep.

<br/>
As an example, in the following I asks the system to rotate immediatly active logs (that cannot be vacuumed while active) and on inactive logs to reduce the size to `10 MB`:


<br/>
<br/>
```shell
% sudo journalctl --rotate --vacuum-size=10M

Vacuuming done, freed 0B of archived journals from /run/log/journal.
Vacuuming done, freed 0B of archived journals from /var/log/journal.
Deleted archived journal /var/log/journal/6d428e8be03e4038b0c1aaabdb440f0f/user-1001@883a7dd1d8c9412fb6e3fc60f8dde880-00000000001d2294-0005bdf47397f855.journal (32.0M).
Deleted archived journal /var/log/journal/6d428e8be03e4038b0c1aaabdb440f0f/system@f57c826bb8fe473b8a350e05b81944d4-00000000001eef78-0005bdfc90bffe8c.journal (96.0M).
Deleted archived journal /var/log/journal/6d428e8be03e4038b0c1aaabdb440f0f/user-1000@3640d3cc286942239ebca805b28ae167-00000000001eef97-0005bdfc90dfde70.journal (32.0M).
...
```
<br/>
<br/>

At the end, the log directory will result in pretty much the specified size, with the exception of active and ongoing log activity.
