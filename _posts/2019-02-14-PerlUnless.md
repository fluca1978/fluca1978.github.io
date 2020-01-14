---
layout: post
title:  "Perl 5 unless"
author: Luca Ferrari
tags:
- perl
- perl
permalink: /:year/:month/:day/:title.html
---
`unless` is a somehow discourage Perl 5 operator, and I have to admit I only have a simple pattern where I find comfortable using it: the *dry-run*.

# Perl 5 `unless`

`unless` is the negation of `if`. Too harsh? But that's the true:

```shell
% perl -E 'say "Hello World!" if ( 1 );'
Hello World!

% perl -E 'say "Hello World!" unless ( 1 );'
```

`unless` usage is generally discouraged because it can make a code block hard to read, especially for non english coders.

<br/>
<br/>
In my applications I often place a flag of *dry-run*, that is a boolean argument to make the application doing all the things it must, except that it does not commit/complete the workflow. I believe `unless` is useful for these kind of situations:

```perl
unless( $opts->dry_run ){
    $statement->execute( @params ) 
       || warn '[ERROR] cannot do stuff against the database!';
}
```

In the above code, I wrap an SQL statement execution with an `unless` conditional that does not pass in the case of the *dry_run* flag set. I cannot use the postfix version, that would result probably in a more readable code, because the line itself contains a postfix exception handling.
