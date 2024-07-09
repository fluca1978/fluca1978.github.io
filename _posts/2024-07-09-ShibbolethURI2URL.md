---
layout: post
title:  "Shibboleth: unable to locate metadata for provider"
author: Luca Ferrari
tags:
- linux
- shibboleth
- nginx
permalink: /:year/:month/:day/:title.html
---
Something I spent hours on...

# Shibboleth: unable to locate metadata for provider

Today I spent a few hours trying to figure out why Shibboleth was not working against our identity provider.
The environment was a Debian 12, something I'm not really used to, but that does not matter.

The Shibboleth version is `3.4.1`, the configuration was migrated from a working machine running Shibboleth version `3.3.0`.

The daemon was loading fine, but when I tried to connect to `/Shibboleth.sso/Login`, I got an error.
The `fastcgi` logs (yes, I'm running against `nginx`) revealed something like:


<br/>
<br/>
```shell
[error] 651#651: *72 FastCGI sent in stderr: "Unable to locate metadata for identity provider (https://foo)"
```
<br/>
<br/>


and the problem was confirmed in the `shibd.log` too:

<br/><br/>
```
WARN Shibboleth.SessionInitiator.SAML2 [1] [default]: unable to locate metadata for provider
```
<br/><br/>


**But the problem was *critical* and was not in the `shibd.log`**!

In fact, inspecting the `shib_warn.log` I found something like:

<br/><br/>
```shell
CRIT Shibboleth.Application : error initializing MetadataProvider: Root of metadata instance not recognized: {urn:mace:shibboleth:3.0:native:sp:config}MetadataFilter
```
<br/><br/>


The solution came from a post on the [mailing list](http://shibboleth.net/pipermail/users/2018-September/041600.html){:target="_blank"} : **chaning the `uri` attribute of the `MetadataProvider` to `url` in the file `shibboleth2.xml` fixed the problem**.

I wonder why such a problematic error is not appearing in every log, including `shibd.log` and why it is so obscure.
