---
layout: post
title:  "PL/Perl now ties %ENV"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A small but great improvement in the security of PL/Perl.

# PL/Perl now ties %ENV

PL/Perl is a great language, since it *ties* together two of my favourite pieces of technology: PostgreSQL and Perl.

While I do usually refer to **PL/Perl** as the capability to run Perl code within PostgreSQL, the correct naming is either **`PL/Perl`** or **`PL/Perlu`**, where the former is the **trusted** language and the latter is the untrusted one.

A trusted language means that the code will run in a PostgreSQL sandbox, with lower permissions than a normal application.

[This commit introduces a new protection level](https://git.postgresql.org/gitweb/?p=postgresql.git;a=commitdiff;h=3ebcfa54d){:target="_blank"} in the trusted languaged `PL/Perl`: is prevents the modification of the `%ENV` hash, that represents the enviromental settings for the running code.
[An official CVE](https://nvd.nist.gov/vuln/detail/CVE-2024-10979){:target="_blank"} for the problem has been issued.



The trick to prevent modifications is really elegant, as often Perl is: using a *tied hash* to wrap up `%ENV`.

It works as follows:
- create a new class that implements the *hash protector*
- `tie` the `%ENV` to this new class
- provide warnings when something tries to modify the `%ENV` tied hash.


Let's explain  it a little better.
First of all, a new class to wrap the hash is created:

<br/>
<br/>
```perl
package PostgreSQL::InServer::WarnEnv;

use strict;
use warnings;
use Tie::Hash;
our @ISA = qw(Tie::StdHash);

sub STORE  { warn "attempted alteration of \$ENV{$_[1]}"; }
sub DELETE { warn "attempted deletion of \$ENV{$_[1]}"; }
sub CLEAR  { warn "attempted clearance of ENV hash"; }

```
<br/>
<br/>

The `PostgreSQL::InServer::WarnEnv` class inherits from `Tie::StdHash`, a tie-able hash that already defines all the required methods and that requires you to only override those that are in your scope of interest.
In particular, `WarnEnv` overrides `STORE`, `DELETE` and `CLEAR` that are method used when adding, deleting of a value in the hash or clearing it all.

Then, it does suffice to `tie` the `%ENV` to this class, and in fact the `PL/Perl` implementation does:

<br/>
<br/>
```perl
tie %main::ENV, 'PostgreSQL::InServer::WarnEnv', %ENV or die $!;
```
<br/>
<br/>

that applies `WarnEnv` as the class behind the behaviour of `main::ENV` keeping all values of `%ENV` (that has been changed to a normal hash).
From now on, trying to modify `%ENV` will result in a warning according to the method used.


This patch has been backported on older PostgreSQL versions until 12.
