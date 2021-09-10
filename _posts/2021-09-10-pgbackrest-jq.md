---
layout: post
title:  "Using jq to get information out of pgbackrest" 
author: Luca Ferrari
tags:
- postgresql
- backup
- planet-postgresql-org
- pgbackrest
permalink: /:year/:month/:day/:title.html
---
pgbackrest supports the JSON output format, and this can be useful to automate some information analysys.

# Using `jq` to get information out of `pgbackrest`

[`pgbackrest`](https://pgbackrest.org/){:target="_blank"} offers the output of its commands in the JSON format. I'm not a great fan of JSON, but it having such an output offers a few advantages, most notably **it is a stable text output format** that can be inspected easily with other tools.
<br/>
In other words, no need for regular expression to parse the textual output, and moreover, the output is guaranteed to be stable, that means no changes will happen (or better, no fields will be removed), while a simple rephrasing in the text output could crash your crafty regular expression!
<br/>
<br/>
Among the available tools, `jq` is a good sheel program that allows you to *parse and navigate* a JSON content.
<br/>
Let's see how it is possible to get some output combining `jq` and `pgbackrest`.

## Get the *last backup* information

When your stanza has a lot of backup, you probably don't want to monitor all of them in deep, but would rather like to get a quick hint on when the last backup did took place.
<br/>
The `pgbackrest info` command reports all the backup available for a given stanza, and it can then be piped into `jq` to get more human readable information.
<br/>
Quick! Show me the snippet:

<br/>
<br/>
```shell
$ pgbackrest info --output json | jq '"Stanza:  " + .[].name + " (" +  .[].status.message + ") " + "Last backup completed at "  +   (.[].backup[-1].timestamp.stop | strftime("%Y-%m-%d %H:%M") )' 

"Stanza:  miguel (ok) Last backup completed at 2021-07-27 09:23"
```
<br/>
<br/>
This is what I would like to see when I'm in a rush and need to see which machine are in trouble with backups: it shows me the name of the stanza, the status of the backup (`ok`) and the time and date the backup ended.
<br/>
Let's analyze the command in more detail:
- `pgbackrest info --output json` enables the output of the `info` command as JSON;
- `jq` is used to parse the JSON output concatenating strings, delimited by `"` with `+`
  - `.[].name` provides the name of the stanza, that is it reads the `name` property of the JSON output;
  - `.[].status.message` provides the backup status message, that is the appearing `ok`;
  - `(.[].backup[-1].timestamp.stop | strftime("%Y-%m-%d %H:%M") )` is clearly the trickiest part, and it gets the last backup (i.e., the backup `-1` from the end), extracts its stop timestamp (there are `start` and `stop` timestamp properties) and filters it (i.e., pipes within `jq`) to `strftime` to display the timestamp in a more human friendly way.


## Get all the backups for a stanza

It is possible to iterate over all the backup information and therefore get an overall status of all the backups:

<br/>
<br/>
```shell
$ pgbackrest info --stanza miguel --output json | jq -r '"Stanza:  " + .[].name + " (" +  .[].status.message + ") " + " backup completed at "  +   (.[].backup[].timestamp.stop | strftime("%Y-%m-%d") ) + " of size " + (.[].backup[].info.size/1024|tostring ) + " MB"' 
Stanza:  miguel (ok)  backup completed at 2021-01-27 of size 3578696.4814453125 MB
Stanza:  miguel (ok)  backup completed at 2021-02-27 of size 3578696.4814453125 MB
Stanza:  miguel (ok)  backup completed at 2021-03-27 of size 3578696.4814453125 MB
Stanza:  miguel (ok)  backup completed at 2021-04-27 of size 3578696.4814453125 MB
Stanza:  miguel (ok)  backup completed at 2021-05-27 of size 3582783.4150390625 MB
Stanza:  miguel (ok)  backup completed at 2021-06-27 of size 3582783.4150390625 MB
Stanza:  miguel (ok)  backup completed at 2021-07-27 of size 3582783.4150390625 MB
Stanza:  miguel (ok)  backup completed at 2021-07-27 of size 3582783.4150390625 MB
Stanza:  miguel (ok)  backup completed at 2021-09-27 of size 3585732.208984375 MB
Stanza:  miguel (ok)  backup completed at 2021-07-27 of size 3585732.208984375 MB
```
<br/>
<br/>
The trick here is to use `-r` to let the application to iterate on every backup information. Also note that it is possible to add the dimension of the backup, as well as other information tailored to your needs.


## Get the *last backup* within a set of servers

It is possible to elaborate a little more on the `jq` extract string and loop it within a simple shell iteration to get information about all your servers. Of course, this is simpler if your servers have all a pre-defined name, like `server-01`, `server-02` and so on.
<br/>


<br/>
<br/>
```shell
$ for server in {1..10}; do printf "Stanza server-%02d with last backup at %s\n" $server "$(  pgbackrest info --stanza $(printf '%02d' $server) --output json |  jq ' (.[0].backup[-1].timestamp.stop | strftime("%Y-%m-%d %H:%M") )' )" ; done
Stanza server-01 with last backup at "2021-07-27 09:23"
Stanza server-02 with last backup at "2021-07-27 01:23"
Stanza server-03 with last backup at "2021-07-27 02:23"
Stanza server-04 with last backup at "2021-07-27 03:23"
Stanza server-05 with last backup at "2021-07-27 05:23"
Stanza server-06 with last backup at "2021-07-27 06:23"
Stanza server-07 with last backup at "2021-07-27 07:23"
Stanza server-08 with last backup at "2021-07-27 08:23"
Stanza server-09 with last backup at "2021-07-27 10:23"
Stanza server-10 with last backup at "2021-07-27 11:23"

```
<br/>
<br/>

Please note the usage of `printf(1)` to cope with numbers like `01`, as well as the `for` to invoke `pgbackrest info` against every single stanza. Similar results can be obtained with `jq` iterations:


# Conclusions

The capability to output information in JSON can simplify a lot the monitoring of the backup status. There is no need to deploy a complex monitoring stack though, and it does suffice to use `jq` to get a report about servers and backups. Of course, being able to navigate the JSON output and play with shell scripting can allow you to get even better results.
