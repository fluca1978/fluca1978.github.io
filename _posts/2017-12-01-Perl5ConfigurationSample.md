---
layout: post
title:  "Perl5 Configuration loop"
author: Luca Ferrari
tags:
- perl
- programming
permalink: /:year/:month/:day/:title.html
---
Any respectable application should have one (or more) configuration files. In this post I explain my idiom to handle configuration in a short and compact way.

# The problem
-----

This is not a tutorial on how to use a configuration system within your application, there are a lot of them out there that are very excellent starting points. In this article I would discuss how I do handle configuration in my small Perl applications in a consistent and quite robust way.

So what is the problem?

The problem is that you applications depends on a few configuration files, at least one, and some of them are **mandatory** while others are **optional**. I don't want the application to continue if a mandatory configuration file does not exist, since it is a call for a break in the application.
On the other hand, how can I configure the application to know which files are mandatory and which are not? This is a *chicken-egg* problem, and the only solution I found is to hardcode this information in the application.
Therefore, I do the following:
1. list *in the application* which configuration files are required, and point out which are mandatory;
2. loop thru the above list to load each configuration file;
3. abort the application if any mandatory file has not been found.

The first step is achievable with a simple hash:

```perl
my $config_files = {
    main => { file => q/config.server.yml/,
              abort_if_missing => 1,
    },
    email_mail => {
        file => q/config.email.yml/,
        abort_if_missing => 0,
    },
    email_auth => {
        file => q/config.email.private.yml/,
        abort_if_missing => 0,
    },

};
```

It's easy enough to see what is happening here: the `abort_if_missing` key describes if the file is mandatory. The keys of the hash are used to avoid processing *not-main files*.

Now I can check the presence of each file and, in case a mandatory file is missing, abort the application:

```perl
for my $config_file ( map { $config_files->{ $_ } } keys %$config_files ){
    next if ( -f $config_file->{ file } );
    die "Required file missing: <$config_file->{ file }>\n"
                 if ( $config_file->{ abort_if_missing } );
    warn "Optional file missing: <$config_file->{ file }>\n"
                if ( ! $config_file->{ abort_if_missing } );
}
```

Now that I'm sure at least the mandatory files are in place, I can load the  configuration. I use `Config::YAML` here, but you can place any config-aware module you like:

```perl
my $configuration = Config::YAML->new( config => $config_files->{ main }->{ file } );
$configuration->read( $_ ) for (
    map { $config_files->{ $_ }->{ file } }
    grep { $_ ne 'main' } keys %$config_files
    );
```

As you can see I read the `main` file first, and then add every *not-main* files. Here I don't check if the file actually exists, since the `Config::YAML` module automatically discards not found files.

## You can do it better

Depending on the configuration module you use, and its capabilities, you can merge all the three steps above into a single one, that mainly performs the loading only if the file exists and aborts at the first mandatory file you don't find. This is not an efficient approach, since it will load possible huge files to just abort the application. A better solution could be to build up a list of only existent configuration files and then load them if all the mandatory files have been found.

Another little improvement could be to not rely on the hash keys to found out the main file, but rather a property in the inner hash. Depending on your application, you could possibly not need at all the distinction between a main file and a not main file, since you can assume that *every mandatory file is also a main file*.

Last but not least, you can group files together: instead of having a per-hash approach, you can use a single hash to describe a group of files.

With all the above assumptions, you can shorten the approach to the following:

```perl
my $config_files = [
    { files => [ qw/config.server.yml/ ],
              mandatory => 1,
    },
    {
        files => [ qw/config.email.yml
            config.email.private.yml/ ],
        mandatory => 0,
    },

    ];


my @to_load;
for my $config_hash ( @$config_files ){
    next if ( ! $config_hash || ! $config_hash->{ files } );
    for my $file_on_disk ( @{ $config_hash->{ files } } ){
        push @to_load, $file_on_disk  if ( -f $file_on_disk );

        die "Config file missing: <$file_on_disk>\n"  if ( ! -f $file_on_disk && $config_hash->{ mandatory } );
        warn "Config file missing: <$file_on_disk>\n" if ( ! -f $file_on_disk );

    }
 }


my $configuration = Config::YAML->new( config => shift @to_load );
$configuration->read( $_ ) for ( @to_load );

```
`
The changes are pretty straigthforward but allow me to explain them in detail:
- the configuration hash changed to an array list of hashes, each sub-hash has a `files` array that include one or more files;
- an array `@to_load` is used to store only those files that can be found on disk, and in the case any of the `files` in a `mandatory` sub-hash is not found the application stops;
- the configuration is then used to load all the found files. Since the files are **all** loaded, and the mandatory files must be there, there is no meaning in the load order.

Therefore, in conclusion, is up to you find out the most reasonable and short **and maintanable** way to load configuration files, but of course Perl offers you ways to handle complex configurations in a few lines of code.`
