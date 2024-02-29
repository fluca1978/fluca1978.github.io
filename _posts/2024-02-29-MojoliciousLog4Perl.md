---
layout: post
title:  "Mojolicious and Log4Perl: an example of configuration"
author: Luca Ferrari
tags:
- mojolicious
- perl
permalink: /:year/:month/:day/:title.html
---
A simple way to dynamically make Log4Perl and Mojolicious to coexist.

# Mojolicious and Log4Perl: an example of configuration

I often use [Log4Perl](https://metacpan.org/pod/Log::Log4perl){:target="_blank"} in my Perl applications, since I'm used to Log4Java since when I was a Java developer.

Mojolicious, on the other hand, being a web framework with almost zero dependencies, does not use Log4Perl, but it is possible to make the two to coexist.
This is Perl after all!

In order to configure Mojolicious to use Log4Perl, there is the need to use the [Mojox::Log::Log4Perl](https://metacpan.org/pod/MojoX::Log::Log4perl){:target="_blank"} plugin, and this is the easy part.

But how to configure Mojolicious to use Log4Perl  **only if it can be configured and only if not in the development process?**
Well, it is easy enough thanks to another plugin that is loaded by default and that is `Mojolicious::Plugin::NotYAMLConfig`: this configuration plugin (as well as its ancestor `Mojolicious::Plugin::JSONConfig`) manages a configuration file as a template, so it is possible to embed Perl code within the configuration.

Therefore, my approach to this is to:
- define all the required settings for Log4Perl in the configuration file of the Mojolicious application;
- add a configuration option that will set a configuration flag to *activate* Log4Perl only if the application is not running under the development server;
- configure the application so that if loads the Log4Perl plugin if and only if the logging is activated and the configuration file is available.

Let's see in details how to achieve this.


## Placing the configuration entries

In the application configuration file I put something like:

<br/>
<br/>
```yaml
log4perl:
  conf   : conf/log4perl.conf
  active : <%= $app->mode eq 'development' ? 0 : 1 %>

```
<br/>
<br/>

The important part is the `active` option, that includes a template expression that will evaluate to `0` if the application mode is `development` (and this is how the mode is set by the development server).
Therefore, when the configuration is loaded, the `log4perl.active` flag will be valued accordingly to the application mode.

## Configuring the application

In the application `startup` method it is possible to get the configuration file name, testing for its presence on the storage and evaluating it if and only if the `active` flag has been evaluated to `1`:

<br/>
<br/>
```perl
# configure Log4Perl
# if possible
my ( $log4perl ) = $config->{ log4perl }->{ active }
    ? $config->{ log4perl }->{ conf } // undef
    : undef;

if ( $log4perl && -f $log4perl ) {
	$self->log->info( "Activating Log4Perl logging via configuration file $log4perl" );
	$self->log( MojoX::Log::Log4perl->new( $log4perl ) );
}

```
<br/>
<br/>

In the `$log4perl` variable I store the configuration file name that comes from the application configuration file.
If the `active` flag is set (i.e., not running under development) and the file name has a value that points to a file on the filesystem, I load the plugin and configure it with the Log4Perl configuration file, otherwise the Mojolicious logging handler remains in place.


# Conclusion

Mojolicious is a very nice and handy web framework with a lot of pieces that allow for dynamic configuration.
