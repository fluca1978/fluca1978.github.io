---
layout: post
title:  "Extract PEC and/or EML attachments"
author: Luca Ferrari
tags:
- linux
permalink: /:year/:month/:day/:title.html
---
How to handle quickly and esaily the extraction of attachments from EML files.

# Extract PEC and/or EML attachments

Sometimes you receive emails that are a little harder than usual to be read.
It's not because they have been written in a different language, rather because your client cannot immediatly display the email content. This happens when you receive an `.eml` file.
<br/>
In Italy, sadly, this is a quite spread behavior because we have something called **PEC**, that stands for **Posta Elettronica Certificata** (Certified email): it is an e-mail sent via specific providers that *handshake* themselves in order to ensure the delivery of the content.
<br/>
Yes, I know, we are now in 2022, and there is no added value, according to me, with this infrastructure. I mean, encrypting and signing emails can do the same, and today's providers are reliable.
<br/>
Anyway, when someone sends you a *PEC* and you don't have a *PEC enabled addressee* mailbox, your email appears as an email with an attached `.eml` file. The odds are that such file contains the attachment you want to get.
<br/>
How to achieve this without involving a complex email client?
<br/>
**`mpack` to the rescue!**
<br/>
`mpack` is a suite of tools to deal with `eml` files, and it can be installed via your distribution package manager, for instance:

<br/>
<br/>

``` shell
% sudo apt install mpack
```
<br/>
<br/>

The most interesting tool of the `mpack` suite, at least with regard to the scope of this article, is `munpack`.
**`mupack`** is a command that reads an `.eml` file and extract every single part (of a multipart message), that is it can extract attachments:

<br/>
<br/>

``` shell
% munpack postacert.eml
Ferrari.pdf (application/pdf)
```
<br/>
<br/>

And that's all folks!
