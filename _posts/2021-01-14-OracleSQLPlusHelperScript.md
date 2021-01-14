---
layout: post
title:  "Oracle SQL Plus Connection Helper Script"
author: Luca Ferrari
tags:
- oracle
- linux
- bash
permalink: /:year/:month/:day/:title.html
---
I created a simple Bash script to ease an interactive connection to Oracle.

Oracle SQL Plus Connection Helper Script
--

Oracle SQL Plus is a **very poor command line client**, at least I believe it does not catch up the Oracle database engine.
<br/>
Anyway, with a *not so little effort* you can get it working on a Linux machine, and for quick and dirty interactions with a database I do prefer the command line utility to the GUI application (e.g., *SQLDeveloper*). Probably I'm biased to the extensive usage of the great PostgreSQL client `psql`!
<br/>
<br/>
One **ugly** thing that Oracle SQL Plus adopts is a **connection string that includes the password (as clear text)** and an ugly syntax to specify the connection parameters, in particular something like:

<br/>
<br/>
```
username/password@server:port/service
```
<br/>
<br/>

Since I'm not good at remember things, I was used to *backward searching* a connection string in my shell history and manipulate it on the fly. But this is boring, so let's create a simple shell script.
<br/>
I decided to implement it in Bash just because I wanted to use a more advanced `prompt` and string substitution than those in the normal shells.
<br/>
Since I'm lazy, I decided that the script should provide me a default for many connection parameters, because most of the time I do connect to a very specific server and I change only the database, ops, the *schema*, I do connect to. Therefore the script has a few *default* variables that contain all my current setup, and asks me only the password and the schema to connect to. However, in those rare cases where I want to connect to another database, I can inform the script to ask me for all the connection settings.
<br/>
<br/>
Enough, let's see the code ([you can download the script here](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/sh/sqlplus.sh){:target="_blank"}):


<br/>
<br/>
```shell
#!/bin/bash

############################################
####   CHANGE ME
############################################
# Change these lines to simplify connection!

ORACLE_DEFAULT_USERNAME="luca"
ORACLE_DEFAULT_ENGINE="test"
ORACLE_DEFAULT_PORT=1521
ORACLE_DEFAULT_SERVER="localhost:$ORACLE_DEFAULT_PORT"


###########################################
#######    SCRIPT STARTS HERE
###########################################


# first of all check we can use sqlplus, otherwise
# it does not make sense to ask the user for
# data.
# The `sqlplus` is looked for into the PATH
# or the ORACLE_HOME folder
SQL_PLUS_EXECUTABLE=$(which sqlplus)
SQL_PLUS_EXECUTABLE=${SQL_PLUS_EXECUTABLE:-$ORACLE_HOME/sqlplus}
if [ ! -x "$SQL_PLUS_EXECUTABLE" ]; then
    echo "Cannot find the executable `sqlplus` ..."
    exit 1
fi


# Ask the user if the default connection properties are fine
if [ ! -z "$ORACLE_DEFAULT_USERNAME" -a ! -z "$ORACLE_DEFAULT_SERVER" -a ! -z "$ORACLE_DEFAULT_ENGINE" ]; then
    CONNECTION_STRING="$ORACLE_DEFAULT_USERNAME@$ORACLE_DEFAULT_SERVER/$ORACLE_DEFAULT_ENGINE"
    echo -en "Does this connection sound good?\n\t$CONNECTION_STRING\n\nOK (Y/n)? "
    read -n 1 answer
    case $answer in
        n|N|no|NO|No|nO)
            echo -e "\nPlease specify the following parameters"
            read -p "Username [$ORACLE_DEFAULT_USERNAME] ? " ORACLE_USERNAME
            read -p "Server [$ORACLE_DEFAULT_SERVER] ? "     ORACLE_SERVER
            read -p "Engine [$ORACLE_DEFAULT_ENGINE]? "      ORACLE_ENGINE
            ;;
        *)
            ;;
    esac

    # set arguments to default or what the user has entered
    ORACLE_USERNAME=${ORACLE_USERNAME:-$ORACLE_DEFAULT_USERNAME}
    ORACLE_SERVER=${ORACLE_SERVER:-$ORACLE_DEFAULT_SERVER}
    ORACLE_ENGINE=${ORACLE_ENGINE:-$ORACLE_DEFAULT_ENGINE}
fi

# here ask for the missing arguments like password and schema
read -p "Oracle Schema (proxy) ? " ORACLE_SCHEMA

# make a proxy user by using the syntax
# username[schema]
#
# e.g., luca[mydb]
#
if [ ! -z "$ORACLE_SCHEMA" ]; then
    ORACLE_USERNAME="$ORACLE_USERNAME[$ORACLE_SCHEMA]"
fi

read -p "$ORACLE_USERNAME password (will not echo chars) ? "    -s ORACLE_PASSWORD
echo



# do the connection!
clear
echo "Connecting $ORACLE_USERNAME@$ORACLE_SERVER/$ORACLE_ENGINE ..."
CONNECTION_STRING="$ORACLE_USERNAME"/"$ORACLE_PASSWORD"@"$ORACLE_SERVER"/"$ORACLE_ENGINE"
$SQL_PLUS_EXECUTABLE $CONNECTION_STRING
```
<br/>
<br/>

What you have to do is to change all the `ORACLE_DEFAULT_xxx` variable at the very beginning of the script.
<br/>
Once the script is launched, it asks you if the connection string with the default arguments is fine. If it is, the script asks you the password and the schema, otherwise it asks you all the connection parameters setting any of them to its default value if you leave it blank.
<br/>
Please note that the script does support *proxy users*, that is an Oracle user that access a different schema. Therefore if you specify a *proxy*, when prompted for it, the system will build a username like `user[schema]`.
<br/>
To make it clear, suppose your username is `luca` and the schema you want to connect to is `mydb`: you need to specify `luca` as `username` and `mydb` as `proxy`. On the other hand, if your username is `mydb` and the schema you want to connect is `mydb`, you have to answer only at the username question and leave unanswered the question on the proxy.
<br/>
<br/>
I hope to enjoy a little more my `sqlplus` experience with this script!
<br/>
Please let me know if you find it useful in any way, also because I'm curious to know how many crazy people are there still using `sqlplus`.
