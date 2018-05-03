---
layout: post
title:  "plperl: which version of Perl?"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
- perl5
permalink: /:year/:month/:day/:title.html
---
`plperl` is a great extension for PostgreSQL that allows the execution of Perl 5 code within the database.

# `plperl`: which version of Perl?

When executing Perl 5 code within the database, PostgreSQL uses the /embedded Perl 5/ to create one (or more) instance of the interpreter. The version of the compiler and virtual machine that runs depends on how PostgreSQL has been compiled, or better, how `libperl.so` has been created.
It is possible to use a specific version of Perl without having to change the system wide Perl 5, and in particular it is possible with some effort to use `perlbrew` to this aim.

## Understanding which `perl` the database is executing

To know which `perl` executable the server will run it is possible to use the `Config` module for a little introspection:

```sql
# DO LANGUAGE plperlu
$PERL$

  use Config;
  elog( INFO, 'Perl executable ' . $Config{ perlpath } );
  elog( INFO, 'Perl version ' . $Config{ version } );
  elog( INFO, 'Perl library ' . $Config{ libperl } );

$PERL$;
```

For example, the above piece of code produces the following output on my system:

```sh
INFO:  Perl executable /usr/local/bin/perl
INFO:  Perl version 5.24.3
INFO:  Perl library libperl.so.5.24.3
```

that tells clearly the `perl` executable is at version `5.24.3`.
The same could have been checked from the `plperl.so` library file, that is linked to the above version of the library:

```sh
% ldd /usr/local/lib/postgresql/plperl.so
/usr/local/lib/postgresql/plperl.so:
        libthr.so.3 => /lib/libthr.so.3 (0x801214000)
        libperl.so.5.24 => /usr/local/lib/perl5/5.24/mach/CORE/libperl.so.5.24 (0x80143c000)
        libc.so.7 => /lib/libc.so.7 (0x800824000)
        libm.so.5 => /lib/libm.so.5 (0x801833000)
        libcrypt.so.5 => /lib/libcrypt.so.5 (0x801a5e000)
        libutil.so.9 => /lib/libutil.so.9 (0x801c7d000)
```

that is `libperl.so.5.24`.


## Installing a different `plperl` version using `perlbrew`

[perlbrew](http://perlbrew.pl) is a great tool to make different versions of Perl 5 co-exist on the same system.
Thanks to `perlbrew** it is possible to install another version of Perl 5 without tossing the system wide installation.
**In order to be usable by PostgreSQL, Perl must be compiled with the shared library option `-D useshrplib`**, so as the user that owns the PostgreSQL daemon:

```sh
% perlbrew install --multi perl-5.26.2 -D useshrplib
...
% perlbrew switch perl-5.26.2
```

It is now possible to compile a new version of PostgreSQL, in order to get the new `plperl`:

```sh
% ./configure --prefix=/opt/postgresql/10/3 --with-perl --with-python --without-readline
...
% ldd /opt/postgresql/10/3/lib/plperl.so 
/opt/postgresql/10/3/lib/plperl.so:
        libperl.so => 
           /var/db/postgres/perl5/perlbrew/perls/perl-5.26.2/lib/5.26.2/amd64-freebsd-multi/CORE/libperl.so 
                                                                                                (0x801214000)
        libc.so.7 => /lib/libc.so.7 (0x800824000)
        libthr.so.3 => /lib/libthr.so.3 (0x801612000)
        libm.so.5 => /lib/libm.so.5 (0x80183a000)
        libcrypt.so.5 => /lib/libcrypt.so.5 (0x801a65000)
        libutil.so.9 => /lib/libutil.so.9 (0x801c84000)
```

As you can see, the `libperl.so` is now linked to the `5.26.2` version of `libperl.so`. 
It is now time to test the new installation!

```sql
# CREATE EXTENSION plperl;
# CREATE LANGUAGE plperlu;

# DO LANGUAGE plperlu
$PERL$

  use Config;
  elog( INFO, 'Perl executable ' . $Config{ perlpath } );
  elog( INFO, 'Perl version ' . $Config{ version } );
  elog( INFO, 'Perl library ' . $Config{ libperl } );

$PERL$;

INFO:  Perl executable /var/db/postgres/perl5/perlbrew/perls/perl-5.26.2/bin/perl
INFO:  Perl version 5.26.2
INFO:  Perl library libperl.so
DO
```

That's it!
