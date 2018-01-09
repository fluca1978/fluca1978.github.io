---
layout: post
title:  "My Perl 6 version of backup_time"
author: Luca Ferrari
tags:
- perl6
- perlsphere
permalink: /:year/:month/:day/:title.html
---

`backup_time.sh` is a simple shell script I use quite often to quick do a single file or whole directory backup via `tar(1)`.
How difficult should it be to write it as a Perl 6 script?

# Introducing `backup_time.p6`

I was used to keep around a few backup scripts to just take copies of various stuff that I would not (or could) place under version control system. Basically, all these scripts worked the same way:
- create an archive of the stuff I care;
- name the archive after the date the backup started;
- place the backup in a *backup place*.

## `backup_time.sh`: the original shell script

Having fighted the lazyness, I decided to write a single shell script, named `backup_time.sh` that performs all the above actions for either a single file or a backup directory. The [script is available on one of my GitHub repos](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/sh/backup_time_file.sh) and simply works as follows:

```shell
backup_time.sh <backup_directory> <file> <dir1> <file2> <dir2> ...
```

The very first argument is the directory where the archives will be placed into, the other (slurpy) arguments are either a single file or a directory to archive. The script performs some simple checks (existent directory, etc.) and then produces a name for a `tar(1)` archive with the timestamp (or something alike) of the backup, so that for a file the name results in `<original_name>-YYYY-mm-DDTHH-MM` while for a directory it is something like `_DIRECTORY_<original_name>-YYYY-mm-DDTHH-MM.tar.bz2`.
Please note that only in the case of a directory a `tar(1)` archive is done: in the case of a single file just a copy of it is performed in order to avoid wasting of resources.

The script does perform some other actions, like computing the *SHA1* hash for the generated archive, as well as removing the oldest copies of the same file archive (or better, the similar named archives).


## `backup_time.p6` the first version of the script

Let's implement a simple Perl 6 script: [here there's `backup-time.p6`](https://github.com/fluca1978/fluca1978-coding-bits/blob/master/perl6/backup_time.p6).
It does not perform exactly the same things the shell counterpart does, but the main engine works. It is not shorter than the shell counterpart, but it is surely shorter than a Perl 5 compatible version mainly due to:
1. the `[IO::Path](https://docs.perl6.org/type/IO::Path)` role;
2. the `[DateTime](https://docs.perl6.org/type/DateTime)` builtin and `where` checks.

Allow me to explain the code line by line.
First of all the `MAIN` declaration:

```perl
sub MAIN( Str :$backup_dir
          where { .IO.d // die "No backup directory [$backup_dir]!" } = 'BACKUP'
    , *@backup_entries where { .map: { .IO.f || .IO.d // die "No backup entry [$_]" }  }
    )
```

The `$backup_dir` is the first argument and must be a directory, or the script could immediatly abort. Here there's the first shrink of code, even if it is not well readable at glance: the `where` code block does perform a check and optionally aborts the script, while the value of `$backup_dir` is automatically set to `BACKUP` in the case no argument is specified.

The `@backup_entries` is a slurpy array (i.e., `*@`), so it catches everything on the command line. Again, a `where` condition allows for a quick check about the existance of the argument as either a directory (`.IO.d`) or a file (`IO.f`) and in the case any of them does not exist the program aborts. Note here that the `map` method is used against the whole array in order to test the whole argument set.

As a sidenote, **while it is true that `where` allows to quickly and early check the arguments provided to the script, it is the `IO::Path` role that makes it possible to really convert them as files/directories** and check for their existance.

The script then reduces to a single `for` loop against the list of arguments:

```perl
    my $now = DateTime.now;
    # for each entry compute the name
    for @backup_entries -> $entry {
        my $archive_name = $backup_dir.IO.add: '%s-%s-%04d-%02d-%02dT%02d%02d'.sprintf(
                                                     ( $entry.IO.d ?? 'DIRECTORY' !! 'FILE' ),
                                                     $entry.IO.basename,
                                                     $now.year,
                                                     $now.month,
                                                     $now.day,
                                                     $now.hour,
                                                     $now.minute );

        "== Backup %s\n\t  [%s]\n\t->[%s]".sprintf( ( $entry.IO.d ?? 'DIRECTORY' !! 'FILE' ), $entry.IO.basename, $archive_name ).say;
        if $entry.IO.d {
            my $current_tar = run 'tar', 'cjvf', $archive_name ~ '.tar.bz2', $entry, :out, :err;
            ( $current_tar.exitcode == 0 ?? 'OK' !! 'KO' ).say;
        }
        else {
            my $ok = $entry.IO.copy( $archive_name );
            ( $ok ?? 'OK' !! 'KO' ).say;
        }
    }
```

The `$archive_name` is a two step made name: first it is created via a `sprintf` call to interpolate the `DateTime` object, prepend the name with either `DIRECTORY` or `FILE` depending on the type of the current `$entry` and the result is `add`ed (i.e., concatenated) to the `$backup_dir` so to build up a full backup name for the archive to be created.

Please note that it could have been possible to reduce the `sprintf` using the `DateTime` embedded formatter at the time of the creation, so something like the following:

```perl
my $now = DateTime.new( formatter => { %04d-%02d-%02dT%02d-%02d'.sprintf: .year, .month, .day, .hour, .minute; } );
...
my $archive_name = $backup_dir.IO.add: '%s-%s-%s'.sprintf( ( $entry.IO.d ?? 'DIRECTORY' !! 'FILE' ),
                                                     $entry.IO.basename, $now );
```
and this is the last version I wrote so far.

After having computed the resulting archive name, it is time to do the real backup.
In the case of a directory the `tar(1)` is executed via the `run` routine (from `Proc`): the archive is appended with the tar suffix and the command is executed; depending on the exit code an *OK/KO* message is printed.

In the case of a file the backup is even simpler, since a `cp(1)` must be performed, and in the case again the `IO::Path` role provides the capabilities to do that via `copy` method to which the final destination is passed as argument. Again, as a result an *OK/KO*  string to inform the user.

## What is missing?

I've not yet implemented the removal of old files, because I'm a bit lazy.
Also the SHA1 computation is still lacking. Now, both of them can be easily performed running external commands, therefore thru `Proc`, but I guess there could be a smarter solution.
