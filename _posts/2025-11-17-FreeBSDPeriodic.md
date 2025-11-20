---
layout: post
title:  "FreeBSD periodic(8)"
author: Luca Ferrari
tags:
- freebsd
permalink: /:year/:month/:day/:title.html
---
What is this periodic-thing named often when dealing with freeBSD?

# FreeBSD periodic(8)

`periodic` is nothing more than a shell script that **wraps** the execution of other scripts.


Sorry to be harsh, but that's it!

Why is so many time cited when dealing with FreeBSD? Well, because the operating system delegates to `periodic` a lot of manainance activities.
But, again, to be straight: **periodic is not a replacement for `cron`**, it is something that **runs based on `cron`**.

First of all, let's inspect the `root` crontab:

```shell
joey# cat /etc/crontab
# /etc/crontab - root's crontab for FreeBSD
#
#
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
#
#minute hour    mday    month   wday    who     command
#
# Save some entropy so that /dev/random can re-seed on boot.
*/11    *       *       *       *       operator /usr/libexec/save-entropy
#
# Rotate log files every hour, if necessary.
0       *       *       *       *       root    newsyslog
#
# Perform daily/weekly/monthly maintenance.
1       3       *       *       *       root    periodic daily
15      4       *       *       6       root    periodic weekly
30      5       1       *       *       root    periodic monthly
...
```

As you can see, `cron` runs three instances of this `periodic` thing with different schedules.

Now, `periodic` lies in `/usr/sbin` and is a shell script that accepts a directory name (`daily`, `weekly`, `monthly`). Those directories are stored into `7etc/periodic` and `/usr/local/etc/periodic`:


```shell
joey# ls /etc/periodic
daily           monthly         security        weekly

joey# ls /usr/local/etc/periodic
daily           security        weekly
```


Within each subdirectory, there are shell scripts that execute tasks:

```shell
oey# ls /etc/periodic/daily
100.clean-disks         222.backup-gmirror      408.status-gstripe      480.status-ntpd
110.clean-tmps          223.backup-zfs          409.status-gconcat      500.queuerun
120.clean-preserve      300.calendar            410.status-mfi          510.status-world-kernel
130.clean-msgs          310.accounting          420.status-network      800.scrub-zfs
140.clean-rwho          400.status-disks        430.status-uptime       801.trim-zfs
150.clean-hoststat      401.status-graid        440.status-mailq        999.local
200.backup-passwd       404.status-zfs          450.status-security
210.backup-aliases      406.status-gmirror      460.status-mail-rejects
221.backup-gpart        407.status-graid3       480.leapfile-ntpd
```


When `periodic` is called, either interactively or via `cron`, and receives a directory argument like in `periodic daily`, it opens the `daily` directory and executes the scripts in a lexicographic order (that's why they are named after a  progressive number).

In order to understand what the scripts do, let's examine one really simple:

```shell
#!/bin/sh
#
#

# If there is a global system configuration file, suck it in.
#
if [ -r /etc/defaults/periodic.conf ]
then
    . /etc/defaults/periodic.conf
    source_periodic_confs
fi

case "$daily_status_uptime_enable" in
    [Yy][Ee][Ss])
        rwho=$(echo /var/rwho/*)
        if [ -f "${rwho%% *}" ]
        then
            echo ""
            echo "Local network system status:"
            prog=ruptime
        else
            echo ""
            echo "Local system status:"
            prog=uptime
        fi
        rc=$($prog | tee /dev/stderr | wc -l)
        if [ $? -eq 0 ]
        then
            [ $rc -gt 1 ] && rc=1
        else
            rc=3
        fi;;

    *)  rc=0;;
esac

exit $rc
```


The script is named `430.status-uptime` and the first (well, not very first) thing it does it to evaluate a variable named `daily_status_uptime_enabled`.
Every `periodic` script is enabled by a variable named after the script (without the leading number) with an underscore substituting every dash and with the `enabled` suffix. Therefore the script `222.backup_gmirror` is enabled via the value `daily_backup_gmirror_enabled`.
Clearly, there are other variables to substitute `daily` to `monthly` and `weekly`.

Once the enabling variable is set to `YES` like `daily_status_uptime_enabled=YES`, the script continues to run and performs its job. In the above example, the script tries to understand the correct `prog`ram to run, e.g., `uptime`. Then it runs the command and returns a normalized valued as follows:
- `0` means the **script did nothing**, e.g., it was not enabled;
- `1` the script produced some output;
- `2` the script has something misconfigured;
- `3` or a greater value, there has been an error.

**Please note: the script has to return `1` in the case of success!**

But where is configured the variable that enables a script?
There is a script `/etc/periodic.conf` that contains a list of variables that enable/disable the set of scripts.
For example, `/etc/defaults/periodic.conf` contains something like:

```shell
# cat /etc/defaults/periodic.conf
...
# 550.ipfwlimit
security_status_ipfwlimit_enable="YES"
security_status_ipfwlimit_period="daily"
...
```

There are other variables that can be set for each script, and that are usually dependent on the script itself.

It is possible to override the default configuration by means of editing `/etc/periodic.conf` or `/usr/local/etc/periodic.conf`.

Logs of the periodic system go into `/var/log/daily` and similar files.
