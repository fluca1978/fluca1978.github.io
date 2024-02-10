---
layout: post
title:  "pgagroal-cli gains JSON output"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- pgagroal
permalink: /:year/:month/:day/:title.html
---
A new feature of `pgagroal-cli` that now makes another step towards the full automation.

# pgagroal-cli gains JSON output

At last, I made it: [a commit in pgagroal to support JSON output](https://github.com/agroal/pgagroal/commit/8b13185c4ea47bf7e09547b8813511db7dd014fd){:target="_blank"}. It has been quite hard and long, not for the technological challenge, rather for all the little details like continuos integration, to get this work completed.
As a rule of thumb, I stated this work last November (of course, slowly working in and out).

What is all of this about?

The idea is to provide JSON based output to `pgagroal-cli`, the command line interface and main management tool for the `pgagroal` connection pool.

I have to admit that **I hate JSON with a passion** and I put it there on the top ranking of my worst formats with XML. So, why did I spent so much time in doing this patch?
If you are not living under a stone, you probably know and see how many tools nowdays provide JSON output format, and the main reason is that this format, while being still human readable (ehm, to some extent!), it allows for an ease automation. There are tons of JSON parsers out there, and even our beloved database PostgreSQL has a very rich JSON support.
Therefore, **having a consistent and automatically parsable command output** wille ease the automation, **and hence the adoption** of `pgagroal`.

To some extent, this work is the natural continuation of the work I initiated almost one year ago to make `pgagroal-cli` command line more consistent and understandable, for example I added commands to handle configuration directly from the command line (see for example [this commit](https://github.com/agroal/pgagroal/commit/ade40240317bad155dbf1e40866c96257b688b90){:target="_blank"}) and to have a more compact and consistent set of commands (see for example this [this commit](https://github.com/agroal/pgagroal/commit/ade40240317bad155dbf1e40866c96257b688b90){:target="_blank"} and the following I made).

## How to use the JSON output

The `pgagroal-cli` command now supports an optional command line flag `--format` that allows to switch from the default text based output to the new JSON format. As the [documentation states](https://github.com/agroal/pgagroal/commit/ade40240317bad155dbf1e40866c96257b688b90){:target="_blank"}, the default output is the text format, so not specifying any `--format` option is totally equivalent to specifying `--format text`.

On the other hand, to turn on the JSON output format, it is required to pass `--format json` on the command line.
As an example, the output of a command will appear to be:

<br/>
<br/>
```shell
% pgagroal-cli ping --format json
{
        "command":      {
                "name": "ping",
                "status":       "OK",
                "error":        0,
                "exit-status":  0,
                "output":       {
                        "status":       1,
                        "message":      "running"
                }
        },
        "application":  {
                "name": "pgagroal-cli",
                "major":        1,
                "minor":        6,
                "patch":        0,
                "version":      "1.6.0"
        }
}

```
<br/>
<br/>

## Format of JSON output

The JSON output has a *fixed* structure with many pre-defined structure that include:
- `command` an object that contains the command the server `pgagroal` has executed (or has been requested to execute);
- `application` reports the name and version of the application that required the command (so far, always `pgagroal-cli`).

The `comamnd` object, in turn, contains other information, like the status of the command and the notification of errors, as well as **`output`, an object that contains the command output (if any) and the command status**.

As an example, consider a more verbose command like `status`:

<br/>
<br/>
```shell
% gagroal-cli status --format json
{
        "command":      {
                "name": "status",
                "status":       "OK",
                "error":        0,
                "exit-status":  0,
                "output":       {
                        "status":       {
                                "message":      "Running",
                                "status":       1
                        },
                        "connections":  {
                                "active":       0,
                                "total":        2,
                                "max":  15
                        },
                        "databases":    {
                                "disabled":     {
                                        "count":        0,
                                        "state":        "disabled",
                                        "list": []
                                }
                        }
                }
        },
        "application":  {
                "name": "pgagroal-cli",
                "major":        1,
                "minor":        6,
                "patch":        0,
                "version":      "1.6.0"
        }
}
```
<br/>
<br/>

As you can see, the `command` has a more extended `output` section that includes much more information and reports, with another dress, the output that the normal text command would have reported.

**Every command has a different `output` format**, that means that in order to interpret every command output there is the need to read the documentation for such command.

Moreover, it is interesting to note that, due to refactoring of the code, **the text command output has slightly changed**, so chances are that if you based your automation on such format you are going to break your scripts. **This is an excellent motivation to switch to the new JSON output format!**

Under the hood, all the complex commands like the above `status` have been refactored to *talk only in JSON*, therefore the text output format is nowdays a purified output extracted from the JSON sent over the communication protocol.


## What about `pgagroal-cli` friends?

The other main command, `pgagroal-admin` has not migrated to JSON deliberately: I don't believe that we need a lot of automation on this command, hence we don't need to provide JSON output. Moreover, the command is not very verbose and does not produce pretty much output, on the other hand it requires an interactive session with the user.

Therefore, I don't see the need to port JSON output to this command.


## A Brief History

This patch is, as often it happens, the result of many trials and errors, either in the implementation or in the design.

In the beginning, I thought to add an explicit `--json` command line flag to indicate the need for JSON output, but I later changed my mind to the more general and extensible `--format` that allows for future addition of output formats, if the need will arise.

I implemented a first prototype using the [json-c](https://github.com/json-c/json-c){:target="_blank"} library, then switched to [cJSON](https://github.com/DaveGamble/cJSON){:target="_blank"}. I have to say that, even if both the libraries deal with JSON, they have a quite different approach in how to build a JSON object. I tend to prefer `cJSON` because it has a less structured approach to add scalar values.

Towards the end of the patch, we had to deal with a lot of issues with the port to OSX, and it required a few days for us to discover that we had not fully updated the `CMakeList.txt` file section related to the OSX part linkage.


# Conclusions

`pgagroal` is growing more and more, and I believe that this new JSON feature will open the road for new exciting developments and integrations with other system, thus promoting the adoption of this tool in the PostgreSQL ecosystem!
