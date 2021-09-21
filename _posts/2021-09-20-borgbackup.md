---
layout: post
title:  "Borg Backup: an introduction"
author: Luca Ferrari
tags:
- backup
- borg
- open source
permalink: /:year/:month/:day/:title.html
---
borg is a great tool for doing backup on *nix machines

# `borg` Backup: an introduction

`borg` is a command line tool to do backups.
<br/>
It has a few itneresting features, most notably:
- it can encrypt your backups before they leave the machine, so that your data is at safe;
- it can deduplicate data stored as backup.

How can `borg` do the deduplication? It separates data incoming from the backup into *chunks*, augmenting therefore the possibility that the same chunk of data is repated over and over across multiple individual files. For every chunk, `borg` stores a single copy of it, thus reducing the required disk space.
<br/>
In other words, it is like `borg` re-arranges the incoming data in a way suitable for *physical storage* instead of its layout that is suitable for *logical storage* (i.e., your filesystem tree).


<br/>
<br/>
As many other command line tools, `borg` has a single entry point executable named, well, `borg`. The command accepts a list of subcommands, each one dedicated to a particular task. You can get a list of commands by typing `borg` alone in a terminal:

<br/>
<br/>
```shell
luca@miguel ~ % borg
usage: borg [-V] [-h] [--critical] [--error] [--warning] [--info] [--debug]
            [--debug-topic TOPIC] [-p] [--log-json] [--lock-wait SECONDS]
            [--bypass-lock] [--show-version] [--show-rc] [--umask M]
            [--remote-path PATH] [--remote-ratelimit RATE]
            [--consider-part-files] [--debug-profile FILE] [--rsh RSH]
            <command> ...

Borg - Deduplicated Backups

...
                                                                                                                                                                                                                                                                                        
  <command>
    mount               mount repository
    serve               start repository server process
    init                initialize empty repository
    check               verify repository
    key                 manage repository key
    change-passphrase   change repository passphrase
    create              create backup
    extract             extract archive contents
    export-tar          create tarball from archive
    diff                find differences in archive contents
    rename              rename archive
    delete              delete archive
    list                list archive or repository contents
    umount              umount repository
    info                show repository or archive information
    break-lock          break repository and cache locks
    prune               prune archives
    upgrade             upgrade repository format
    recreate            Re-create archives
    with-lock           run user command with lock held
    config              get and set configuration values
    debug               debugging command (not intended for normal use)
    benchmark           benchmark command

```
<br/>
<br/>

`borg` requires you to specify a *repository*, that is where you are going to store your backups. A repository could be an external hard drive, a partition mounted on your filesystem, or a remote filesystem accessed via SSH.

<br/>
Whithin a repository, `borg` manages *archives*, that are the actual backups. I often refer to an archive as a *label* in the following, because it seems to me clearer.

## Backups and Repository

The backup is identified by two pieces of information:
- an URL that identifies where the backups are stored (e.g., a remote machine), namely the *repository*;
- a backup label (the *archive*), separated from the URL by a double colon (`::`), that identifies a single backup set.
<br/>
As an example, a backup is identified by `/backup/documents::MY_DOCUMENTS_2021-09-06`, where:
- `/backup/documents` is the local path on the filesystem to the backup repository;
- `MY_DOCUMENTS_2021-09-06` is the label that identifies a specific backup archive within such repository.
<br/>
Another example could be `ssh://backup@carmensita/backup/borg/miguel::miguel-2021-09-06` where:
- `ssh://backup@carmensita/backup/borg/miguel` is a path on a remote host named `carmensita` that can be reached by means of SSH;
- `miguel-2021-09-06` is the backup label on the remote backup set.
<br/>
You can avoid typing the repository URL by setting the `BORG_REPO` environment path, so that when `borg` finds a backup label where a complete URL is expected it will try to expand such variable. In other words, the backup label `::miguel-2021-09-06` typed by its own will be treated as if you typed `$BORG_REPO::miguel-2021-09-06`, and if that provides a valid backup identification the `borg` application will run smoothly.






# `borg` in action

In the following, I'm assuming there are two machine to work with:
- `carmensita` is the backup machine, that is the remote location where the backups are going to be stored;
- `miguel` is the target machine, that is the machine I want to backup.

Both the machine are Linux based, for what it matters. The backups will be launched from the target machine, that is `miguel`, since as far as I know there is no way to start a backup remotely.

## Installation

On the Fedora machine, installing `borg` was as simple as:

<br/>
<br/>
```shell
luca@carmensita ~ % sudo dnf install borgbackup
```
<br/>
<br/>

while I had a few issues on trying to install the package `borg-backup` package on CentOS.
Therefore I decided to install the portable executable of `borg`:

<br/>
<br/>
```shell
luca@miguel ~ % fetch https://github.com/borgbackup/borg/releases/download/1.1.17/borg-linux64
luca@miguel ~ % chmod 755 borg-linux64
luca@miguel ~ % sudo mv borg-linux64 /usr/bin/borg
```
<br/>
<br/>


## Ensure SSH keys are in place

I want to be able to the backup from the user `luca` on the `miguel` machine to the backup machine `carmensita` using the `backup` account on that machine, therefore I need to exchange the key from `miguel` to `carmensita`.
This is quite simple to do:

<br/>
<br/>
```shell
luca@miguel ~ % ssh-copy-id -i .ssh/id_rsa.pub backup@carmensita
```
<br/>
<br/>

So far, there is no need to ensure the opposite is true, therefore `carmensita` is not able to connect to `miguel` by means of the `backup` user.



## Initializing the repository

I decided to place my backup repository on the local filesystem `/backup/borg/miguel` on the `carmensita` host, so I need to create the backup path on the remote backup host or I can use the `borg` command to make it do for me.
<br/>
This is the command used to initialize the backup repository:

<br/>
<br/>
```shell
luca@miguel ~ % borg init -e repokey --progress \
                          --make-parent-dirs    \
                          ssh://backup@carmensita/backup/borg/miguel
                          
Enter new passphrase: 
Enter same passphrase again: 
Do you want your passphrase to be displayed for verification? [yN]: N
                                                                                                                                                   
By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://backup@carmensita/backup/borg/miguel

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).

```
<br/>
<br/>
Let's inspect the above command to see what it does:
- `init` tells `borg` to create the directory structure to hold backup data;
- `ssh://backup@carmensita/backup/borg/miguel` is the repository location. In this case I use SSH specifying the user (`backup`) and the backup machine (`carmensita`) as well as the local filesystem where I want the backups to be stored at;
- `--progress` shows some progress if needed;
- `--make-parent-dirs` instructs `borg` to create the whole directory tree on the backup machine if not already there. In this particular case, it allows me to skip to connect to `carmensita` and run `mkdir -p /backup/borg/miguel` manually;
- `-e repokey` specifies the encryption mode, that is to use a per-repository key that will be stored within the backup repository itself.

<br/>
<br/>
Once the initialization phase has completed, on the backup machine there will be the diretory structure ready to accept the backup content, as well as the configuration of the repository contined into the `config` file:

<br/>
<br/>
```shell
luca@carmensita ~ % sudo ls /backup/borg/miguel
config  data  hints.1  index.1  integrity.1  nonce  README

luca@carmensita ~ % sudo cat /backup/borg/miguel/config
[repository]
version = 1
segments_per_dir = 1000
max_segment_size = 524288000
append_only = 0
storage_quota = 0
additional_free_space = 0
id = c84b695c57ed6438fc87905d1fc65037f704524b92a815bd366053c2abaf82b4
key = hqlhbGdvcml0aG2mc2hhMjU2pGRhdGHaAN4zKfFnfHqEzOtALP4cwxVtmiZpZpUfMZYC0v
      ...
                        

```
<br/>
<br/>

As you can see, the configuration file contains the `key` of the repository, since I created the repository with the `repokey` encryption method. **The repository key is stored along with the repository**, therefore as the program suggest you to, you need to backup the key somewhere safe (e.g., on an ecrypted storage somewhere else) in the case you need to later use for recovering the repository.
<br/>
In order to get a backup copy of the repository key, you need to use the `key` command with the `export` subcommand, specifying the repository URL and the file you want to contain the backup into (e.g., `miguel-repo-carmensita.key`).

<br/>
<br/>
```shell
luca@miguel ~ % borg key export \
                     ssh://backup@carmensita/backup/borg/miguel \
                     miguel-repo-carmensita.key
                     
luca@miguel ~ % cat miguel-repo-carmensita.key 
BORG_KEY c84b695c57ed6438fc87905d1fc65037f704524b92a815bd366053c2abaf82b4
hqlhbGdvcml0aG2mc2hhMjU2pGRhdGHaAN4zKfFnfHqEzOtALP4cwxVtmiZpZpUfMZYC0v
```
<br/>
<br/>

As you can see, the key file is a simple text file that contains an *header* that identifies it as a `BORG_KEY`, followed by the **repository id** and on the second line the exact same key stored on the repository configuration file.
<br/>
Keep such file safe, along with the passphrase you have chosen, in order to be able to access to the repository.



## Using an environment variable to avoid typing the `borg` repository

This is not mandatory, but you can use a shell environment variable to avoid typing a long repository URL every time you need to issue a `borg` command. In the following, wherever you see `BORG_REPO` you can imagine the same `ssh://backup@carmensita/backup/borg/miguel` string is placed:


<br/>
<br/>
```shell
luca@miguel ~ % export BORG_REPO=ssh://backup@carmensita/backup/borg/miguel
```
<br/>
<br/>

You can use whatever you want as an environment variable name, but `borg` will automatically search for a variable named `BORG_REPO` and, if it finds it, such variable will be used when no explicit repository URL is typed. This means that you can omit `$BORG_REPO` at all in your command lines. As an example, the following two commands are equivalent:

<br/>
<br/>
```shell
% borg list $BORG_REPO::miguel-2021-09-01T07-02
...
% borg list ::miguel-2021-09-01T07-02
```
<br/>
<br/>
but please note that in the latter there is no repository URL, just a backup label with the double colon prefix (`::`). Whenever you see `$BORG_REPO` you can safely omit it.

## Checking the repository status

So far, I did not have any backup, but I can check the repository status to look for errors or warnings. The `check` command does that:

<br/>
<br/>
```shell
luca@miguel ~ % borg --info check $BORG_REPO
Remote: Starting repository check
Remote: Starting repository index check
Remote: Index object count match.
Remote: Completed repository check, no problems found.
Starting archive consistency check...
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
Archive consistency check complete, no problems found.

```
<br/>
<br/>

The `--info` option causes `borg` to be a little more verbose, so to see what it is doing on the repository identified by the `$BORG_REPO` variable.
<br/>
Please note that enar the end of the check, the program is asking for the passphrase of the repository, and that is because `borg` is going to check my backups (that so far are missing), and in order to do so it needs to decrypt the data stored.


## Doing the first backup

It is now time to push something to the backup repository. Let's start with a common folder, the `~/Documents`, that contains a bunch of files of different type:

<br/>
<br/>
```shell
luca@miguel ~ % ls -1hs Documents                                                 
totale 71M
3,7M document_1.txt
 64M document_2.txt
4,0K document_3.txt
1,8M gnuemacsref.png
1,4M gnu-wp2.png
132K Luca_Ferrari.pdf

```
<br/>
<br/>

In order to create a new backup you need to use the **`create`** command (there is no a `backup` command, and I don't know why!). 
<br/>
Every backup must have the repository URL and an **archive name** that is a label you want to do to the backup. For example, I can label the backup with the machine name and the time the backup was taken by using the `hostname` and `date` commands:

<br/>
<br/>
```shell
luca@miguel ~ % borg create --progress --stats \
                     $BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M') \
                     ~/Documents
                     
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
------------------------------------------------------------------------------                                                                     
Archive name: miguel-2021-09-01T07-02
Archive fingerprint: 13cc96addc8949b73b454b2492562f4a602a3ffc14d01cd19df150b888f99f8a
Time (start): Wed, 2021-09-01 07:02:33
Time (end):   Wed, 2021-09-01 07:02:36
Duration: 3.68 seconds
Number of files: 6
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               45.66 MB             45.81 MB             45.81 MB
All archives:               45.66 MB             45.81 MB             45.81 MB

                       Unique chunks         Total chunks
Chunk index:                      29                   29
------------------------------------------------------------------------------

```
<br/>
<br/>
The command works as follows:
- `create` tells `borg` to do a new backup;
- `$BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M')` expands to `ssh://backup@carmensita/backup/borg/miguel::miguel-2021-09-01T07-02`, that is the repository name, a `::` and the backup label, as reported on the `Archive name` output line;
- `--progress` tells `borg` to show what it is doing while it is doing;
- `--stats` prints out the final table where you can get a glance at the deduplication level, the size of the backup and other *on-disk* information for this backup;
- `~/Documents` is the path I want to backup. It is possible to specify multiple paths at once, just appending a new entry to the command line.

<br/>
Before doing the actual backup, the system asks me for the passphrase in order to cipher the data.
<br/>
<br/>
Note that the output reports the total amount required to do the backup and the number of files.
<br/>
As you can see, the final backup requires around `45 MB` of disk space, both *plain* and *deduplicated*, and that is because the data has been converted into `29 chunks` that are *unique*, that is not redundant and thus not deduplicable.
<br/>
<br/>
On the backup machine, the space occupied by the backup is the same as reported by `borg`:

<br/>
<br/>
```shell
luca@carmensita ~ % sudo du -hs /backup/borg/miguel/
44M     /backup/borg/miguel/
```
<br/>

## Doing a second backup

Let's remove a file and do a new backup, and see what happens:

<br/>
<br/>
```shell
luca@miguel ~ % mv ~/Documents/document_1.txt my_first_document.txt

luca@miguel ~ % borg create --stats \ 
                $BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M') \
                ~/Documents 
                
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
------------------------------------------------------------------------------
Archive name: miguel-2021-09-01T07-11
Archive fingerprint: 83bbb3dc03ce58db906433f2bdd6ea11bbe8a41043d46adb90e778bb8dcb0529
Time (start): Wed, 2021-09-01 07:12:00
Time (end):   Wed, 2021-09-01 07:12:02
Duration: 2.20 seconds
Number of files: 5
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               41.81 MB             41.95 MB              2.16 kB
All archives:               87.47 MB             87.76 MB             45.81 MB

                       Unique chunks         Total chunks
Chunk index:                      31                   56
------------------------------------------------------------------------------

```
<br/>
<br/>
This time `borg` tells me that the amount of data that I wanted to backup was `41 MB`, while the deduplicated size was `2 kB`, and that is because I haven't changed anything but removed a file. As you can see, out of `51 chunks` there are `31` unique, that is `borg` has been applying deduplication this time. The end result, is that `borg` tells me that *all* the backups so far require `45 MB` of disk space, as confirmed by the backup machine:


<br/>
<br/>
```shell
luca@carmensita ~ % sudo du -hs /backup/borg/miguel/
44M     /backup/borg/miguel/

```
<br/>
<br/>


## Change the content and do a backup

Let's do a third backup after having added a file and deleted another file:

<br/>
<br/>
```shell
luca@miguel ~ % mv my_first_document.txt Documents 
luca@miguel ~ % rm Documents/Luca_Ferrari.pdf 

luca@miguel ~ % borg create --stats \
                     $BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M') \
                     ~/Documents
                     
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
------------------------------------------------------------------------------
Archive name: miguel-2021-09-01T07-17
Archive fingerprint: cd007da51579f94414da946c0c7f8461f5e21a01152b2223ed6c667e777103fc
Time (start): Wed, 2021-09-01 07:17:48
Time (end):   Wed, 2021-09-01 07:17:50
Duration: 1.99 seconds
Number of files: 5
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               45.53 MB             45.71 MB              2.20 kB
All archives:              133.00 MB            133.46 MB             45.82 MB

                       Unique chunks         Total chunks
Chunk index:                      33                   84
------------------------------------------------------------------------------


```
<br/>
<br/>

Again, the final deduplication is huge because we placed back content was already there from the very beginning and removed other content.
And again, the disk usage on the backup machine is as `borg` report as `Deduplicated size` on the `All archives` line:

<br/>
<br/>
```shell
luca@carmensita ~ % sudo du -hs /backup/borg/miguel/
44M     /backup/borg/miguel/

```
<br/>
<br/>


## Let's screw a file and do another backup

Now let's change totally a single file and see what happens in the backup:

<br/>
<br/>
```shell
luca@miguel ~ % dd if=/dev/random of=Documents/document_2.txt bs=1024 count=50000 
dd: warning: partial read (82 bytes); suggest iflag=fullblock
0+50000 record dentro
0+50000 record fuori
3849522 bytes (3,8 MB, 3,7 MiB) copied, 8,74478 s, 440 kB/s

```
<br/>
<br/>

The `document_2.txt` is changed in both size and content, so *it is a totally different file* right now. Let's do a backup:

<br/>
<br/>
```shell
luca@miguel ~ % borg create --stats \
    $BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M') \
    ~/Documents
    
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
------------------------------------------------------------------------------
Archive name: miguel-2021-09-01T07-21
Archive fingerprint: 5663b7829dc872df43b86528d799e77836fe21199f3993ceff500f88efd8c2f6
Time (start): Wed, 2021-09-01 07:23:04
Time (end):   Wed, 2021-09-01 07:23:06
Duration: 1.52 seconds
Number of files: 5
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               10.88 MB             10.92 MB              3.87 MB
All archives:              143.88 MB            144.38 MB             49.68 MB

                       Unique chunks         Total chunks
Chunk index:                      38                   95
------------------------------------------------------------------------------

```
<br/>
<br/>
This time the backup size corresponds pretty much to the size of the newly added data, since it has not been deduplicated (because it has been generated randomly), and the archive size on the backup machine increased to `49 MB`.


## Inspect the archive content

How can I know which backups are available for a restoration?
<br/>
The `list` command comes to the rescue:


<br/>
<br/>
```shell
luca@miguel ~ % borg list $BORG_REPO
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
miguel-2021-09-01T07-02              Wed, 2021-09-01 07:02:33 [13cc96addc8949b73b454b2492562f4a602a3ffc14d01cd19df150b888f99f8a]
miguel-2021-09-01T07-11              Wed, 2021-09-01 07:12:00 [83bbb3dc03ce58db906433f2bdd6ea11bbe8a41043d46adb90e778bb8dcb0529]
miguel-2021-09-01T07-17              Wed, 2021-09-01 07:17:48 [cd007da51579f94414da946c0c7f8461f5e21a01152b2223ed6c667e777103fc]
miguel-2021-09-01T07-21              Wed, 2021-09-01 07:23:04 [5663b7829dc872df43b86528d799e77836fe21199f3993ceff500f88efd8c2f6]

```
<br/>
<br/>

There are four different backups in the repository. It is also possible to inspect the content of a specific backup by specifying it after the repository:


<br/>
<br/>
```shell
luca@miguel ~ % borg list $BORG_REPO::miguel-2021-09-01T07-21
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
drwxrwxr-x luca   luca          0 Wed, 2021-09-01 07:17:35 home/luca/Documents
-rw-rw-r-- luca   luca    3849302 Wed, 2021-09-01 06:53:27 home/luca/Documents/my_first_document.txt
-rw-rw-r-- luca   luca    3849522 Wed, 2021-09-01 07:21:16 home/luca/Documents/document_2.txt
-rw-rw-r-- luca   luca         11 Wed, 2021-09-01 06:50:06 home/luca/Documents/document_3.txt
-rw-r--r-- luca   luca    1783280 Wed, 2021-09-01 06:50:59 home/luca/Documents/gnuemacsref.png
-rw-r--r-- luca   luca    1394752 Wed, 2021-09-01 06:50:59 home/luca/Documents/gnu-wp2.png

```
<br/>
<br/>

The above means that I can restore the backup content with the files shown in the listing.

## Searching for a particular file to be restored

I have deleted the `Luca_Ferrari.pdf` file before the backup labeled as `miguel-2021-09-01T07-17`, and that means I can get the file back if I restore either the backup `miguel-2021-09-01T07-02` or `miguel-2021-09-01T07-02`.
<br/>
But in real life, I will not remember when I did canceled a file, so how can I search it back?
<br/>
There is always the solution to mock some shell scripting to do the trick, for example looping thru all the archives and searching for a particular file (or a pattern):

<br/>
<br/>
```shell
luca@miguel ~ % for a in $( borg list $BORG_REPO --short ); do borg list --short $BORG_REPO::$a | grep Luca_Ferrari.pdf && echo " -> $a "; done
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
home/luca/Documents/Luca_Ferrari.pdf
 -> miguel-2021-09-01T07-02 
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
home/luca/Documents/Luca_Ferrari.pdf
 -> miguel-2021-09-01T07-11 
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 
Enter passphrase for key ssh://backup@carmensita/backup/borg/miguel: 

```
<br/>
<br/>

but as you can see, the above command is going to ask the passphrase for every single backup, and this makes the task a little boring. Even if there is the chance to automate the passphrase selection, the above is a poor-man-solution.
<br/>
There is a trick to avoid placing the passphrase every time a repository operation must be done: **using the `BORG_PASSPHRASE` environment variable prevents `borg` to ask you for such a passphrase**. While this is a great way to automate your scripts, it is a way I don't like very much because it makes the passphrase *visibile* to other applications and (possibly) users. However, for the record, this is what happens if you use such variable and repeat the command above:


<br/>
<br/>
```shell
luca@miguel ~ % export BORG_PASSPHRASE=my_strong_and_complicated_passphrase!!!
luca@miguel ~ % for a in $( borg list $BORG_REPO --short ); do borg list --short $BORG_REPO::$a | grep Luca_Ferrari.pdf && echo " -> $a "; done
home/luca/Documents/Luca_Ferrari.pdf
 -> miguel-2021-09-01T07-02 
home/luca/Documents/Luca_Ferrari.pdf
 -> miguel-2021-09-01T07-11 
```
<br/><br/>
and as you can see, there is no request for a passphrase.


## Mounting a backup for introspection and inspection

`borg` allows you to *mount* a backup archive as a regular filesystem, so that you can inspect it and navigate as an ordinary filesystem to search for files and diff them against the current versions you have.
<br/>
The command to do this is, surprisingly, `mount`, and it requires a *mount point* and a backup label. The mounting is done by means of *FUSE (Filesystem USEr space)*, so you need to have `fuse` installed on your machine (e.g., on Fedora and RedHat based `dnf install fuse`).
<br/>
Let's create a mountpoint and attach a backup to see its content:


<br/>
<br/>
```shell
luca@miguel ~ % mkdir borg_mnt
luca@miguel ~ % borg mount ::miguel-2021-09-01T07-02 borg_mnt
# equivalent to
# borg mount $BORG_REPO::miguel-2021-09-01T07-02 borg_mnt
luca@miguel ~ % ls borg_mnt/home/luca/Documents 
document_1.txt  document_2.txt  document_3.txt  gnuemacsref.png  gnu-wp2.png  Luca_Ferrari.pdf

```
<br/>
<br/>

The documents are there, and I can inspect them to see what are the differences, or restore them, or do whatever I would do with files. In short, `borg mount` allows you to have a glance at previous versions just like a *filesystem snapshot* would do.
<br/>
As an example, if I quickly `diff` the directories I can see which files have been removed, which have changed and so on:


<br/>
<br/>
```shell
luca@miguel ~ % diff borg_mnt/home/luca/Documents ~/Documents 
Solo in borg_mnt/home/luca/Documents: document_1.txt
I file binari borg_mnt/home/luca/Documents/document_2.txt e /home/luca/Documents/document_2.txt sono diversi
Solo in borg_mnt/home/luca/Documents: Luca_Ferrari.pdf
Solo in /home/luca/Documents: my_first_document.txt

```
<br/>
<br/>

The mounted filesystem appears as a *borgfs* device of type *fuse*, and does not appear in regular disk based commands:

<br/>
<br/>
```shell
luca@miguel ~ % mount
...
borgfs on /home/luca/borg_mnt type fuse (ro,nosuid,nodev,relatime,user_id=1002,group_id=1003,default_permissions)
luca@miguel ~ % df -h
File system          Dim. Usati Dispon. Uso% Montato su
devtmpfs             467M     0    467M   0% /dev
tmpfs                485M   16K    485M   1% /dev/shm
tmpfs                485M  6,6M    479M   2% /run
tmpfs                485M     0    485M   0% /sys/fs/cgroup
/dev/mapper/cl-root   17G   16G    1,2G  94% /
/dev/sda1           1014M  246M    769M  25% /boot
tmpfs                 97M     0     97M   0% /run/user/1002
luca@miguel ~ % 

```
<br/>
<br/>
Also note that the filesystem is mountde in *read-only* mode (`ro`): you can navigate the filesystem but cannot change a backup set.
<br/>
Once you are done with your introspection, you can detach the filesystem with `borg umount`:

<br/>
<br/>
```shell
luca@miguel ~ % borg umount borg_mnt
```
<br/>
<br/>

You can mount more than one backup at the same time, and it is quite easy to automate the mounting of a backup with something like the following:


<br/>
<br/>
```shell
luca@miguel ~ % for a in $( borg list $BORG_REPO --short ); do mkdir $a && borg mount ::$a $a; done                                            

luca@miguel ~ % mount
...
borgfs on /home/luca/miguel-2021-09-01T07-02 type fuse (ro,nosuid,nodev,relatime,user_id=1002,group_id=1003,default_permissions)
borgfs on /home/luca/miguel-2021-09-01T07-11 type fuse (ro,nosuid,nodev,relatime,user_id=1002,group_id=1003,default_permissions)
borgfs on /home/luca/miguel-2021-09-01T07-17 type fuse (ro,nosuid,nodev,relatime,user_id=1002,group_id=1003,default_permissions)
borgfs on /home/luca/miguel-2021-09-01T07-21 type fuse (ro,nosuid,nodev,relatime,user_id=1002,group_id=1003,default_permissions)

```
<br/>
<br/>

Now it is possible to search for a specific file againt all the backups and get a glance at *when* things changed:


<br/>
<br/>
```shell
luca@miguel ~ % find miguel-* -name document_2.txt -print0 | xargs -0 ls -1hs
 37M miguel-2021-09-01T07-02/home/luca/Documents/document_2.txt
 37M miguel-2021-09-01T07-11/home/luca/Documents/document_2.txt
 37M miguel-2021-09-01T07-17/home/luca/Documents/document_2.txt
3,7M miguel-2021-09-01T07-21/home/luca/Documents/document_2.txt

```
<br/>
<br/>

Apparently, `document_2.txt` has never been removed nor changed until the last backup, when its size was dramatically reduced.
<br/>
<br/>
The point is: thanks to `borg mount` you can apply any of your favourite filesystem level tools to mess around with backup copies.

<br/>
To automatically umount all the entries you can issue a command like the following:


<br/>
<br/>
```shell
luca@miguel ~ % for a in $( mount | grep borgfs | cut -f 3 -d ' ' ); do borg umount $a; done
```
<br/>
<br/>

or use whatever you'd like to unmount FUSE filesystems.


## Finding the differences

While mounting a backup set can be great to exploit your own tools to find out differences, `borg` includes a specific command named `diff` to show you a report about differences in archives. The command allows you to compare only two archives at a time, and they must reside on the same backup machine.
<br/>
As an example:


<br/>
<br/>
```shell
luca@miguel ~ % borg diff ::miguel-2021-09-01T07-02 miguel-2021-09-01T07-21 
  +3.8 MB  -38.5 MB home/luca/Documents/document_2.txt
added       3.85 MB home/luca/Documents/my_first_document.txt
removed     3.85 MB home/luca/Documents/document_1.txt
removed   132.28 kB home/luca/Documents/Luca_Ferrari.pdf

```
<br/>
<br/>
The command shows what happened: a file has been added and two were removed.
<br/>
Please note that the second argument to the command is just a simple name, not a backup URL: it does not have the leading `::` in front of it.


## Extracting a backup

Once we have decided that we want to restore a single backup, `borg` provides us with different choices.
<br/>
First of all, we can extract a *portable* backup, that is something we can work and migrate to a different machine using file system tools. This is done with the `export-tar` subcommand:


<br/>
<br/>
```shell
luca@miguel ~ % borg export-tar ::miguel-2021-09-01T07-21 latest.tar
luca@miguel ~ % tar tvf latest.tar 
drwxrwxr-x luca/luca         0 2021-09-01 07:17 home/luca/Documents/
-rw-rw-r-- luca/luca   3849302 2021-09-01 06:53 home/luca/Documents/my_first_document.txt
-rw-rw-r-- luca/luca   3849522 2021-09-01 07:21 home/luca/Documents/document_2.txt
-rw-rw-r-- luca/luca        11 2021-09-01 06:50 home/luca/Documents/document_3.txt
-rw-r--r-- luca/luca   1783280 2021-09-01 06:50 home/luca/Documents/gnuemacsref.png
-rw-r--r-- luca/luca   1394752 2021-09-01 06:50 home/luca/Documents/gnu-wp2.png

``**
<br/>
<br/>

Simple, sweet and to the point.
<br/>
**Please be careful: the generated `tar` file is not encrypted!** Exporting backup sets can make your data at danger.
<br/>
<br/>
What if we want to restore a backup set? Imagine I've deleted totally my `~/Documents` folder and now I want it back:


<br/>
<br/>
```shell
# DANGER!
luca@miguel ~ % rm -rf ~/Documents

luca@miguel ~ % borg extract ::miguel-2021-09-01T07-21

luca@miguel ~ % ls ~/Documents         
ls: impossibile accedere a '/home/luca/Documents': No such file or directory
```
<br/>
<br/>

Ops! Where are my files now?
<br/>
Similar to what happens to `tar`, the backup contains relative paths, so my files are into `~/home/luca/Documents`.
This can be clearer with the `--list` option, that forces `borg` to tell me what it is extracting:

<br/>
<br/>
```shell
luca@miguel ~ % borg extract ::miguel-2021-09-01T07-21 --list
home/luca/Documents
home/luca/Documents/my_first_document.txt
home/luca/Documents/document_2.txt
home/luca/Documents/document_3.txt
home/luca/Documents/gnuemacsref.png
home/luca/Documents/gnu-wp2.png

```
<br/>
<br/>

In order to demonstrate the power of `extract`, let's change the `~/Documents` folder so that it contains a subfolder with *private* data:


<br/>
<br/>
```shell
# restore content from the extracted backup
luca@miguel ~ % mv home/luca/Documents ~/Documents 
luca@miguel ~ % mkdir ~/Documents/PRIVATE
luca@miguel ~ % echo "My password" > ~/Documents/PRIVATE/password.txt

```
<br/>
<br/>

Now, let's do a new backup of the current folder content:


<br/>
<br/>
```shell
luca@miguel ~ % borg create --stats \
    $BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M') \
    ~/Documents
------------------------------------------------------------------------------
Archive name: miguel-2021-09-08T06-18
Archive fingerprint: 9db1e39b4ab7530f4b951b1b4b94675407b313155ea2f5dd69e2c94c9504f09c
Time (start): Wed, 2021-09-08 06:18:28
Time (end):   Wed, 2021-09-08 06:18:29
Duration: 0.79 seconds
Number of files: 6
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:               10.88 MB             10.92 MB              1.60 kB
All archives:              154.76 MB            155.31 MB             49.68 MB

                       Unique chunks         Total chunks
Chunk index:                      41                  107
------------------------------------------------------------------------------

```
<br/>
<br/>

The backup has been labeled as `miguel-2021-09-08T06-18`.
<br/>
Imagine we want to extract only the `PRIVATE` subfolder of the backup, not the whole backup content. This is possible by means of *patterns* that pretty much every `borg` command support. Be aware that not all commands support the same context of patterns: some commands accept file related patterns, other path patterns.
<br/>
In this case, to get out from the backup only the `PRIVATE` subfolder:

<br/>
<br/>
```shell
luca@miguel ~ % borg extract --list  ::miguel-2021-09-08T06-18 re:PRIVATE
home/luca/Documents/PRIVATE
home/luca/Documents/PRIVATE/password.txt

```
<br/>
<br/>

What is that `re` in front of the pattern?
<br/>
Allow me to briefly explain how patterns work in `borg`.

### Patterns

`borg` supports different pattern *styles*, and a style is identified by a short name separated from the real pattern by a colon (`:`).
<br/>
In the previous example the pattern was written as `re:PRIVATE`, where:
- `re` is the pattern style, that means *regular expression*;
- `PRIVATE` is the pattern, that is the regular expression to match.

<br/>
<br/>
Do you want to extract only those files ending with `png` by a regular expression? The regular expression becaomes `re:"\.png$"` to indicate whatever has a period and ends in `png`.
<br/>
Another popular pattern matching style is the shell one, identified by `sh:`. However, this pattern matching requires more accurate patterns, so the above *PNG only* regular expression would be `sh:"home/luca/*/*.png"` because you need to specify the path as a shell would understand.
<br/>
<br/>
There are other pattern matching *engines* and syntaxes, for more information refer to the help `borg help patterns`. The interesting part is that you can also specify multiple patterns to **include** or **exclude** from operations and **you can store patterns into a file** so that you don't have to specify them every time on the command line.
<br/>
So for example, you can create a text file with all your patterns on a single line, where a `+` means *include* and a `-` means *exclude*:

<br/>
<br/>
```shell
luca@miguel ~ % cat >> my_backup_list.txt
- sh:/home/luca/Documents/PRIVATE
+ re:\.txt$
- re:\.png$


```
<br/>
<br/>
and then you can launch, for example, a backup with the option `pattern-from`:

<br/>
<br/>
```shell
luca@miguel ~ % borg create --patterns-from my_backup_list.txt \
                           --list \
                           $BORG_REPO::$(hostname)-$(date +'%Y-%m-%dT%H-%M') \
                           ~/Documents
x /home/luca/Documents/PRIVATE
x /home/luca/Documents/PRIVATE/password.txt
U /home/luca/Documents/my_first_document.txt
U /home/luca/Documents/document_2.txt
U /home/luca/Documents/document_3.txt
x /home/luca/Documents/gnuemacsref.png
x /home/luca/Documents/gnu-wp2.png
d /home/luca/Documents

```
<br/>
<br/>
The `PRIVATE` folder and its content, as well as all files ending with `.png` have been excluded from the backup, as marked by the `x** on the left of their line, while other files have been inserted.
<br/>
**ORDER MATTERS! If an addind rule comes before the exclude rule, it can win!**
<br>
Therefore, using the following content of the pattern file:

<br/>
<BR/>
```shell
luca@miguel ~ % cat >> my_backup_list.txt
- re:\.png$
+ re:\.txt$
- sh:/home/luca/Documents/PRIVATE

```
<br/>
<br/>
the file `PRIVATE/my_password.txt` would have been added to the backup because it is a file ending in `.txt` and such files are added by means of the very first rule.


## Retention

No backup system is complete without a retention mechanism. A retention policy instruments the backup software about how many backups to keep, trashing the others in order to avoid an infinte disk space consumption.
<br/>
`borg` allows you to `prune` old backups according to a retention time policy: you can specify how many sets to keep for month, day, hour or even second, and of course as a number of backups without any regard to their timeline.
<br/>
In order to make it clear, let's keep the *last week* only backups: the command is `prune` and there are a few `--keep` options to specify the time range:

<br/>
<br/>
```shell
luca@miguel ~ % borg list
miguel-2021-09-01T07-02              Wed, 2021-09-01 07:02:33 [13cc96addc8949b73b454b2492562f4a602a3ffc14d01cd19df150b888f99f8a]
miguel-2021-09-01T07-11              Wed, 2021-09-01 07:12:00 [83bbb3dc03ce58db906433f2bdd6ea11bbe8a41043d46adb90e778bb8dcb0529]
miguel-2021-09-01T07-17              Wed, 2021-09-01 07:17:48 [cd007da51579f94414da946c0c7f8461f5e21a01152b2223ed6c667e777103fc]
miguel-2021-09-01T07-21              Wed, 2021-09-01 07:23:04 [5663b7829dc872df43b86528d799e77836fe21199f3993ceff500f88efd8c2f6]
miguel-2021-09-08T06-18              Wed, 2021-09-08 06:18:28 [9db1e39b4ab7530f4b951b1b4b94675407b313155ea2f5dd69e2c94c9504f09c]
miguel-2021-09-08T06-57              Wed, 2021-09-08 06:57:46 [38e45ba4ee92194d37d74413efc720c160c2a63e553f54238bfbfa1f18ea5679]
miguel-2021-09-08T06-58              Wed, 2021-09-08 06:58:08 [d5e32aa03ff87f35fdc13aef5e93b45cbe6c352581d1d1cee81c4bc986e9055b]
miguel-2021-09-08T06-59              Wed, 2021-09-08 06:59:33 [6b269b030b4d0e6a904b09d65c09fdcd014ce3d479b763490fb8eed957d59f42]
miguel-2021-09-08T07-00              Wed, 2021-09-08 07:00:54 [b6e90c396a683b04616333eb0db53b7a8773f4a3b1f6f5a522ca8e2f212fe001]
miguel-2021-09-08T07-02              Wed, 2021-09-08 07:02:01 [93e40f24132d66455ff01a40a613d624c545706206182fddb23d83e4c407c0b3]
miguel-2021-09-08T07-03              Wed, 2021-09-08 07:03:13 [634b659f9f1ea2756d95fc0dc37ec047e266af07e5e2e19942d9cc8f451c1ea2]
miguel-2021-09-08T07-04              Wed, 2021-09-08 07:04:06 [04c455e6d166234fd088dcb913d48468c63a98b893c364cf9170d38881184a48]
miguel-2021-09-08T07-05              Wed, 2021-09-08 07:05:03 [379585259d534a2a5c57e50aaf58dd6f552cfbc52e6d98441ec7e0dd94307398]
miguel-2021-09-08T07-08              Wed, 2021-09-08 07:08:26 [54b0cdc741fb89c7ad9c7d0b185ca7fa0de99933a91a9a49c30eb34b0475c2a7]
miguel-2021-09-08T07-09              Wed, 2021-09-08 07:09:18 [ef1382cebc277d80902b7d3ccf00b5d6bcf8d6fba6343b6cf4de1b365f1898b8]
miguel-2021-09-08T07-10              Wed, 2021-09-08 07:10:05 [e5c283ab5a971503f5d0f8d3833574ca6f653a632be55e7e7d6d810e86f982b8]



luca@miguel ~ % borg prune --keep-weekly 1

luca@miguel ~ % borg list                 
miguel-2021-09-08T07-10              Wed, 2021-09-08 07:10:05 [e5c283ab5a971503f5d0f8d3833574ca6f653a632be55e7e7d6d810e86f982b8]

```
<br/>
<br/>
The command has deleted all the backups outside the current week, keeping only one backup on the current week (the last).
It is possible to get an hint at what `borg` is doing by means of the `--list` option. Here's another example to make it clear that only one backup within the current week will be kept:

<br/>
<br/>
```shell
luca@miguel ~ % borg list                                                                                                 
miguel-2021-09-08T07-41              Wed, 2021-09-08 07:41:11 [636e4c29e0cb9b1b2e63bec25db0490e8416e3cf7639258a56c09753506e911d]
miguel-2021-09-08T07-42              Wed, 2021-09-08 07:42:09 [87e24d3be4dc574a1ae7732e27b6e76929af086e4e87710c96d345656de0e31b]

luca@miguel ~ % borg prune --list --keep-weekly 5
Keeping archive: miguel-2021-09-08T07-42              Wed, 2021-09-08 07:42:09 [87e24d3be4dc574a1ae7732e27b6e76929af086e4e87710c96d345656de0e31b]
Pruning archive: miguel-2021-09-08T07-41              Wed, 2021-09-08 07:41:11 [636e4c29e0cb9b1b2e63bec25db0490e8416e3cf7639258a56c09753506e911d] (1/1)

```
<br/>
<br/>
In this case the pruning has been specified as `5`, that means keep the last *five weeks, one backup per week*.
<br/>
**The `--dry-run` option can be very useful in this cases to see what `borg` is going to remove from the backup sets**, especially when you start to mix different policies:


<br/>
<br/>
```shell
luca@miguel ~ % borg prune --list --keep-weekly 5 --keep-minutely 3  --dry-run
Keeping archive: miguel-2021-09-08T07-46              Wed, 2021-09-08 07:46:08 [45ed8ff68efbdc4911530813804b94092f10ab4be341c40a7dd63a94dc1598b7]
Keeping archive: miguel-2021-09-08T07-45              Wed, 2021-09-08 07:45:28 [18fd61f563d804092724f617e88a1ff2b913546b48cfe092f965bf24b8083eb7]
Keeping archive: miguel-2021-09-08T07-44              Wed, 2021-09-08 07:44:49 [27050caee6e3cf3b5839b041024fb3599062751f1b537ee291a260426dc891d4]
Would prune:     miguel-2021-09-08T07-43              Wed, 2021-09-08 07:43:26 [1b0d6cc85ddbe8193979ed84b3195b46a18ee93046fc12d2e9c78a0d2a78f88d]

```
<br/>
<br/>
in the above we state to keep last five weeks and no more than three backups with a difference within a minute.
<br/>
There is also a way to specify to keep the last backups, without any regard to their timestamp, and this is done via the `--keep-last` option:


<br/>
<br/>
```shell
luca@miguel ~ % borg prune --list --keep-last 2 --dry-run                   
Keeping archive: miguel-2021-09-08T07-53              Wed, 2021-09-08 07:53:12 [c8bdf9bd6f8569f87f1bd85fa5f22d70e989e784a3c16c477c678301257294e7]
Keeping archive: miguel-2021-09-08T07-46              Wed, 2021-09-08 07:46:08 [45ed8ff68efbdc4911530813804b94092f10ab4be341c40a7dd63a94dc1598b7]
Would prune:     miguel-2021-09-08T07-45              Wed, 2021-09-08 07:45:28 [18fd61f563d804092724f617e88a1ff2b913546b48cfe092f965bf24b8083eb7]
Would prune:     miguel-2021-09-08T07-44              Wed, 2021-09-08 07:44:49 [27050caee6e3cf3b5839b041024fb3599062751f1b537ee291a260426dc891d4]
Would prune:     miguel-2021-09-08T07-43              Wed, 2021-09-08 07:43:26 [1b0d6cc85ddbe8193979ed84b3195b46a18ee93046fc12d2e9c78a0d2a78f88d]
```
<br/>
<br/>
and only the most two recent backups will survive. You can also mix and match the quantity and timeline retention:

<br/>
<br/>
```shell
luca@miguel ~ % borg prune --list --keep-last 1  --keep-minutely 3 --dry-run
Keeping archive: miguel-2021-09-08T07-53              Wed, 2021-09-08 07:53:12 [c8bdf9bd6f8569f87f1bd85fa5f22d70e989e784a3c16c477c678301257294e7]
Keeping archive: miguel-2021-09-08T07-46              Wed, 2021-09-08 07:46:08 [45ed8ff68efbdc4911530813804b94092f10ab4be341c40a7dd63a94dc1598b7]
Keeping archive: miguel-2021-09-08T07-45              Wed, 2021-09-08 07:45:28 [18fd61f563d804092724f617e88a1ff2b913546b48cfe092f965bf24b8083eb7]
Keeping archive: miguel-2021-09-08T07-44              Wed, 2021-09-08 07:44:49 [27050caee6e3cf3b5839b041024fb3599062751f1b537ee291a260426dc891d4]
Would prune:     miguel-2021-09-08T07-43              Wed, 2021-09-08 07:43:26 [1b0d6cc85ddbe8193979ed84b3195b46a18ee93046fc12d2e9c78a0d2a78f88d]
```
and in this case four backups are kept: the last one (`--keep-last 1`) and the three within the difference in minutes (`--keep-minutely 3`).
<br/>

Imagine now the backup sets are as follows (please note that there are two different days):

<br/>
<br/>
```shell
luca@miguel ~ % borg list
miguel-2021-09-08T07-43              Wed, 2021-09-08 07:43:26 [1b0d6cc85ddbe8193979ed84b3195b46a18ee93046fc12d2e9c78a0d2a78f88d]
miguel-2021-09-08T07-44              Wed, 2021-09-08 07:44:49 [27050caee6e3cf3b5839b041024fb3599062751f1b537ee291a260426dc891d4]
miguel-2021-09-08T07-45              Wed, 2021-09-08 07:45:28 [18fd61f563d804092724f617e88a1ff2b913546b48cfe092f965bf24b8083eb7]
miguel-2021-09-08T07-46              Wed, 2021-09-08 07:46:08 [45ed8ff68efbdc4911530813804b94092f10ab4be341c40a7dd63a94dc1598b7]
miguel-2021-09-08T07-53              Wed, 2021-09-08 07:53:12 [c8bdf9bd6f8569f87f1bd85fa5f22d70e989e784a3c16c477c678301257294e7]
miguel-2021-09-10T02-55              Fri, 2021-09-10 02:55:11 [b2def013cc07ed7bcdb239dc2653723579441a835b6de1469d65c65b2c172037]
miguel-2021-09-10T02-58              Fri, 2021-09-10 02:58:34 [d4aa88711e144f7644742a1caa2d03343554c643391ec587dec5db9ecd58c760]

```
<br/>
<br/>

In such above situation, keeping 2 backups *daily* will make `borg` pruning all but last backups in every day:

<br/>
<br/>
```shell
luca@miguel ~ % borg prune --list   --keep-daily 2 --dry-run
Keeping archive: miguel-2021-09-10T02-58              Fri, 2021-09-10 02:58:34 [d4aa88711e144f7644742a1caa2d03343554c643391ec587dec5db9ecd58c760]
Would prune:     miguel-2021-09-10T02-55              Fri, 2021-09-10 02:55:11 [b2def013cc07ed7bcdb239dc2653723579441a835b6de1469d65c65b2c172037]
Keeping archive: miguel-2021-09-08T07-53              Wed, 2021-09-08 07:53:12 [c8bdf9bd6f8569f87f1bd85fa5f22d70e989e784a3c16c477c678301257294e7]
Would prune:     miguel-2021-09-08T07-46              Wed, 2021-09-08 07:46:08 [45ed8ff68efbdc4911530813804b94092f10ab4be341c40a7dd63a94dc1598b7]
Would prune:     miguel-2021-09-08T07-45              Wed, 2021-09-08 07:45:28 [18fd61f563d804092724f617e88a1ff2b913546b48cfe092f965bf24b8083eb7]
Would prune:     miguel-2021-09-08T07-44              Wed, 2021-09-08 07:44:49 [27050caee6e3cf3b5839b041024fb3599062751f1b537ee291a260426dc891d4]
Would prune:     miguel-2021-09-08T07-43              Wed, 2021-09-08 07:43:26 [1b0d6cc85ddbe8193979ed84b3195b46a18ee93046fc12d2e9c78a0d2a78f88d]

```
<br/>
<br/>

As already mentioned, **I strongly recommend doing some `--dry-run` execution before pruning your backups**, since the `borg` pruning rules could be a little different from what you may think about.
<br/>
The `keep-xxly` options keep **up to the specified number of backups**. Just to make it more clear:
- `--keep-hourly 7` will keep the most recent backup on an hour, up to seven total backup, that can be read as "keep the most recent backup per hour up to 7";
- `--keep-daily 3` will keep the most recent backups on every day up to a maximum of three, that is it can be read as "keep the most recent backups on the last three days".

The number specifed as an argument represents the *maxmimum number of backup sets to keep*, according to the policy specified.


## Restoring the repository key

As already explained, if the repository is encrypted (a strongly recommended option), the backup data (i.e., the data contained in the backup sets) will not be available if the repository does not own the key.
<br/>
The key is stored into the repository `config` file, as already shown. In the accidental case the `config` file is corrupted, the repository will not be able to work.
<br/>
In such scenario, the `borg import` command can help, assuming you have a backup copy of the repository key done by means of `borg export`, and you should have done this as soon as the repository has been created.
<br/>
The procedure to repair a corrupted repository is simple, first of all let's check if the repository is crewed:

<br/>
<br/>
```shell
luca@miguel ~ % borg list                                       
Traceback (most recent call last):

  File "/usr/lib64/python3.9/configparser.py", line 789, in get
    value = d[option]

  File "/usr/lib64/python3.9/collections/__init__.py", line 941, in __getitem__
    return self.__missing__(key)            # support subclasses that define __missing__

  File "/usr/lib64/python3.9/collections/__init__.py", line 933, in __missing__
    raise KeyError(key)

KeyError: 'key'


During handling of the above exception, another exception occurred:


Traceback (most recent call last):

  File "/usr/lib64/python3.9/site-packages/borg/remote.py", line 247, in serve
    res = f(**args)

  File "/usr/lib64/python3.9/site-packages/borg/repository.py", line 331, in load_key
    keydata = self.config.get('repository', 'key')

  File "/usr/lib64/python3.9/configparser.py", line 792, in get
    raise NoOptionError(option, section)

configparser.NoOptionError: No option 'key' in section: 'repository'

Borg server: Platform: Linux carmensita 5.10.11-200.fc33.x86_64 #1 SMP Wed Jan 27 20:21:22 UTC 2021 x86_64
Borg server: Linux: Unknown Linux  
Borg server: Borg: 1.1.17  Python: CPython 3.9.1 msgpack: 0.5.6.+borg1
Borg server: PID: 188693  CWD: /home/backup
Borg server: sys.argv: ['/usr/bin/borg', 'serve', '--umask=077']
Borg server: SSH_ORIGINAL_COMMAND: None
Platform: Linux miguel 4.18.0-305.12.1.el8_4.x86_64 #1 SMP Wed Aug 11 01:59:55 UTC 2021 x86_64
Linux: CentOS Linux 8.4.2105 
Borg: 1.1.17  Python: CPython 3.7.11 msgpack: 0.5.6.+borg1
PID: 94795  CWD: /home/luca
sys.argv: ['borg', 'list']
SSH_ORIGINAL_COMMAND: None

```
<br/>
<br/>
Ok, the repository is no more accessible at all (I've removed the key from the configuration file!), so let's repair it:

<br/>
<br/>
```shell
luca@miguel ~ % borg key import  $BORG_REPO miguel-repo-carmensita.key 

luca@miguel ~ % borg list
miguel-2021-09-08T07-43              Wed, 2021-09-08 07:43:26 [1b0d6cc85ddbe8193979ed84b3195b46a18ee93046fc12d2e9c78a0d2a78f88d]
miguel-2021-09-08T07-44              Wed, 2021-09-08 07:44:49 [27050caee6e3cf3b5839b041024fb3599062751f1b537ee291a260426dc891d4]
miguel-2021-09-08T07-45              Wed, 2021-09-08 07:45:28 [18fd61f563d804092724f617e88a1ff2b913546b48cfe092f965bf24b8083eb7]
miguel-2021-09-08T07-46              Wed, 2021-09-08 07:46:08 [45ed8ff68efbdc4911530813804b94092f10ab4be341c40a7dd63a94dc1598b7]
miguel-2021-09-08T07-53              Wed, 2021-09-08 07:53:12 [c8bdf9bd6f8569f87f1bd85fa5f22d70e989e784a3c16c477c678301257294e7]
miguel-2021-09-10T02-55              Fri, 2021-09-10 02:55:11 [b2def013cc07ed7bcdb239dc2653723579441a835b6de1469d65c65b2c172037]
miguel-2021-09-10T02-58              Fri, 2021-09-10 02:58:34 [d4aa88711e144f7644742a1caa2d03343554c643391ec587dec5db9ecd58c760]

```
<br/>
<br/>
and the repository is *online* again!
<br/>
Please note that the repairing process has been done from the target machine, that is *remotely*!


# Conclusions

I think `borg` is a very powerful tool, and I'm progressively switching all my backups into `borg` related repositories and archives.
<br/>
What I like the most is the capability to encrypt and deduplicate the backups.
Moreover, being a command line tool with pre-defined commands makes it easy to automate by means of shell scripting and already existing scheduling tools like `cron`.
<br/>
<br/>
However, I don't feel comfortable with the terminology used by `borg`. It is, of course, a matter of taste, but I don't understand why the command to launch a backup is `create` and not a simple `backup`. Also I don't like the name *archive*, since to me is much more a backup label (I could be biased to Bacula here).
<br/>
Also the `prune` command is not very clear in my opinion, for different reasons. First, the `--keep` set of options are somehow difficult to reason about, at least in my opinion. It is not simple to build a retention policy that is clear. But most notably, I do believe that pruning should be done automatically as soon as a backup completes. In other words, deleting old backups should not be an option left to the user (and here I'm biased by `pgbackrest`). Last, `prune` is a bad name according to me, while `expire` is a better choice.
<br/>
<br/>
Last but not least, having the capability to store the repository passphrase into an environment variable sounds to me awful. While it can be useful in a protected environment, it could expose the whole system at a danger! Here, I agree with the OpenSSH way of thinking: don't let any way to store users' passwords. Probably combining the environment variable with something a little more robust, like `pass` can be a better choice for a more secure environment.

<br/>
<br/>
Besides the matter of taste and habit, `borg` is a great solution for doing backups at an enterprise level, and I hope to see a lot of adoption.
