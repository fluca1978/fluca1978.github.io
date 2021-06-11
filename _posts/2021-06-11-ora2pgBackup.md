---
layout: post
title:  "Using ora2pg to do a kind of backup" 
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
How I implemented a kind of Oracle-to-PostgreSQL backup.

# Using ora2pg to do a kind of backup

**Disclaimer: `ora2pg` is an amazing tool, but is not supposed to be used as a backup tool!**
<br/>
In this article I'm going to show you how I decided to implement a kind of *Oracle-to-PostgreSQL* backup by means of [`ora2pg`](https://ora2pg.darold.net/){:target="_blank"}.
<br/>
<br/>
It all started as a simple need: *migrate an Oracle database to PostgreSQL to do some experiments*.
<br/>
Therefore I fired up an `ora2pg` project, and started from there in order to do the migration.
<br/>
End of the story.
<br/>
But then I was asked to migrate *again* the same database, because in the meantime something changed.
<br/>
And then again, and again.
<br/>
I'm not saying I was asked to keep the database synchronized, but to sometime load an updated amount of data (and structures) from Oracle to PostgreSQL.
<br/>
As lazy as I am, after a couple of request I was producing a simple shell script to automate the job, at least about running `ora2pg`. Yes, this could be less trivial than you think, since `ora2pg` relies on the Oracle instaclient to be installed (with all the environment set), and Perl to be ready with all the `DBD::Oracle`, `DBI` and other stuff in the right place. And this is a little complicated on my machines because I tend to experiment, and so I have a lot of different stuff installed, so I have to fire up the right Perl, with the right modules, and the right environment (I do use [`perlbrew`](https://perlbrew.pl){:target="_blank"}, in the case you are wondering). In other words, there was some setup work necessary before I could run `ora2pg`, and that was a perfect candidate for a real shell script.
<br/>
Then, the number of the databases to do this work on became two, and this was a call for a parametric script...you get the point!
<br/>
Last but not least, I was not sure about when the migration would happen and when I was asked to load a new bunch of stuff into PostgreSQL, and since my memory is lazier than me, I not always do remember all the required steps to load the extracted part of `ora2pg` into our beloved database.
<br/>
And therefore I decided to write a simple shell script to allow me to:
- extract data from a customizable Oracle database, assuming `ora2pg` project is configured;
- place the data and structures in a well defined space on my storage;
- create a compact and clear `psql` script to load the extraction into PostgreSQL (yes, I know `ora2pg` can do this automagically with a PostgreSQL connection, but I have to do it offline).
<br/>
<br/>
Let's start first from how I do add new databases to my script:

<br/>
<br/>
```shell
export ORACLE_HOME=/opt/oracle/instantclient_18_3
source ~/perl5/perlbrew/etc/bashrc
DATE_DIR=`date '+%Y-%m-%d'`
BACKUP_ROOT=/backup
ORACLE_PG_TEMPALTE="my_oracle_template"

do_ora2pg ora-srv ORADB1 /backup/ora2pg/ORADB1
do_ora2pg ora-srv ORADB2  /backup/ora2pg/ORADB2

```
<br/>
<br/>
The initial part is used to set up Perl and Oracle Instant Client.
<br/>
The `do_ora2pg` are the lines that define a single extraction; the arguments to the `do_ora2pg` shell function are:
- Oracle host name;
- Oracle schema to which I need to connect;
- path to the `ora2pg` project.

<br/>
<br/>
What does the `do_ora2pg` shell function do?
<br/>
Here it is:

<br/>
<br/>
```shell
do_ora2pg()
{
    local SERVER_NAME=$1
    local ORACLE_SCHEMA=$2
    local ORACLE_PROJECT_FOLDER=$3

    local BACKUP_DIR="${BACKUP_ROOT}/${SERVER_NAME}/${DATE_DIR}/ora2pg/${ORACLE_SCHEMA}"
    local PG_DATABASE=$( echo $ORACLE_SCHEMA | tr '[:upper:]' '[:lower:]' )

    if [ ! -d "$BACKUP_DIR" ]; then
        mkdir -p "$BACKUP_DIR"
    else
        rm $BACKUP_DIR/*.sql > /dev/null 2>&1
    fi

    echo -e "\n{ $ORACLE_SCHEMA }\n\t=> PostgreSQL Dump in [$BACKUP_DIR]\n"

    cd $BACKUP_DIR


    TYPES="TABLE VIEW MVIEW INSERT SEQUENCE FUNCTION PROCEDURE TRIGGER"
    counter=1

    cat <<EOF > all.sql
-- Automatic PostgreSQL reload from Oracle
-- $ORACLE_SCHEMA
-- $BACKUP_DIR

\set ON_ERROR_STOP 1
\set QUIET         1

\echo Reload of Oracle schema $PG_DATABASE

DROP DATABASE IF EXISTS $PG_DATABASE;
CREATE DATABASE $PG_DATABASE WITH TEMPLATE $ORACLE_PG_TEMPLATE
\c $PG_DATABASE


CREATE EXTENSION IF NOT EXISTS orafce;


\echo Connected to $PG_DATABASE
\echo Starting the loading batch
\echo

EOF

    for t in $TYPES
    do
        output=$(printf "%02d" $counter)-$t.sql
        echo "$t => $output "
        echo -e "\n\\\echo Batch to load : $output ..." >> all.sql
        ora2pg --c ${ORACLE_PROJECT_FOLDER}/config/ora2pg.conf  -t $t -o $output >> ora2pg.log 2>&1
        if [ $? -eq 0 ]; then
            echo "OK"
            echo -e "\\\i $output" >> all.sql
        else
            echo "KO"
            echo  -e "\\\echo NOT LOADING!" >> all.sql
        fi

        counter=$(( counter + 1 ))
    done

    
}

```
<br/>
<br/>
The function initially creates a `BACKUP_DIR` that is named after a well defined root, and after the date the backup is took on (I assume to do no more than one per day). The idea is that the backup directory will result in something like `/backup/ora-srv/2021-06-11/db1`.
After a quick check about the existance or not of the backup directory, the script creates a file named `all.sql` in such directory, placing some `psql` directives into such file.
<br/>
Then the script executes `ora2pg` for the objects I care about, producing a different file name suffix for every kind of invocation, for example `01-TABLES` for table structures (schema).
If the dump of the objects type is fine, the `\i` inclusion of that file is placed into `all.sql`, otherwise an alert is inserted.
<br/>
The result `all.sql` is a file like the following:

<br/>
<br/>
```sql
-- Automatic PostgreSQL reload from Oracle
-- ORADB1
-- /backup/DATI/ora-srv/2021-06-10/ora2pg/ORADB1

\set ON_ERROR_STOP 1
\set QUIET         1

\echo Reload of Oracle schema db1

DROP DATABASE IF EXISTS db1;
CREATE DATABASE db1 WITH TEMPLATE my_oracle_template;
\c db1


CREATE EXTENSION IF NOT EXISTS orafce;


\echo Connected to db1
\echo Starting the loading batch
\echo


\echo Batch to load : 01-TABLE.sql ...
\i 01-TABLE.sql

\echo Batch to load : 02-VIEW.sql ...
\i 02-VIEW.sql

\echo Batch to load : 03-MVIEW.sql ...
\i 03-MVIEW.sql

\echo Batch to load : 04-INSERT.sql ...
\i 04-INSERT.sql

\echo Batch to load : 05-SEQUENCE.sql ...
\i 05-SEQUENCE.sql

\echo Batch to load : 06-FUNCTION.sql ...
\i 06-FUNCTION.sql

\echo Batch to load : 07-PROCEDURE.sql ...
\i 07-PROCEDURE.sql

\echo Batch to load : 08-TRIGGER.sql ...
\i 08-TRIGGER.sql

```
<br/>
<br/>
Therefore, the only thing I have to do when I want to *migrate* the Oracle content into PostgreSQL, is to launch a command like:

<br/>
<br/>
```shell
% psql -U luca template1 < all.sql
```
<br/>
<br/>
and wait. This is something easy enough for me to remember even if I have not sleep well!
<br/>
I've experimented with this for a few weeks now, and it is something that is really useful to my use case.
<br/>
Please note that I create the extension `orafce` in the reloaded database, because we do use some functions that are dumped and reloaded well by this extension. For that reason, the database on the PostgreSQL side is created by means of a specific template that have the extension already installed.


# Conclusions 
`ora2pg` is an amazing tool, that can be used and abused in different ways including doing backups!
<br/>
I'm sure there are smarter ways to achieve my same aim, and I will report back if I learn about them, so please let me know if you have suggestions!
