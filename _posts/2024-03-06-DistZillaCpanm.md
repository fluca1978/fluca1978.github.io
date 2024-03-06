---
layout: post
title:  "Dist::Zilla and App::cpanminus to quickly install all dependencies"
author: Luca Ferrari
tags:
- perl
- distzilla
permalink: /:year/:month/:day/:title.html
---
A quick trick to ensure all the dependencies for a project are in place

# Dist::Zilla and App::Cpanm to quickly install all dependencies

I tend to use `Dist::Zilla` as distribution *builder* and `App::cpanminus` as my *package manager*.

The former has a very useful command, `listdeps` that scans your project source code and lists all the dependencies; the latter is a very straightforward package manager for Perl.

As an example, here's the output for a project of mine:

<br/>
<br/>
```shell
% dzil listdeps
base
Config::YAML
DateTime
DBIx::Class::Core
DBIx::Class::Schema
Excel::Writer::XLSX
experimental
ExtUtils::MakeMaker
File::Path
File::Spec
JSON
lib
List::Util
Log::Log4perl
Mojo::File
Mojo::JSON
Mojo::URL
Mojo::UserAgent
Mojo::Util
Moo
namespace::clean
overload
Spreadsheet::Reader::ExcelXML
strict
Test::More
Time::Duration
Time::HiRes
utf8
warnings
XML::LibXML

```
<br/>
<br/>


It is possible to combine the two commands, so to ensure that all your dependencies are installed and at the right version:

<br/>
<br/>
```shell
% dzil listdeps | cpanm
```
<br/>
<br/>

and that's it!

This is particularly useful when you upgrade (or change version) Perl, so to ensure that everything on your working tree will continue to work.
