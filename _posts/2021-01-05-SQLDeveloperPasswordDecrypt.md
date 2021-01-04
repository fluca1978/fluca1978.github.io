---
layout: post
title:  "Decrypting password stored in Oracle SQL Developer"
author: Luca Ferrari
tags:
- oracle
- python
- java
permalink: /:year/:month/:day/:title.html
---
Is it possible to get it back the encrypted passwords?

Decrypting password stored in Oracle SQL Developer
---

Oracle SQL Developer is a Java client used to connect to Oracle databases. The application does allow you to store several connections, each one with a password.
<br/>
The password are stored in an encrypted form into an XML file, that on Unix like operating systems is `~/.sqldeveloper/system4.1.5.21.78/o.jdeveloper.db.connection.12.2.1.0.42.151001.541/connections.xml` where `12.2.1.0.42.151001.541` is the SQL Developer version. In other words, the user customized connections are stored as XML into the `connections.xml` file under the `~/.sqldeveloper/system<version>/o.jdeveloper.db.connection.<version>`.
<br/>
In that file, there is a `<StringRefAddr>` tag that contains the encrypted string for the password, so for example:

<br/>
<br/>
```xml
<StringRefAddr addrType="password">
  <Contents>XyzAbcedf...+7tFuoM</Contents>
</StringRefAddr>
```
<br/>
<br/>

In particular, all the XML file is made by tags of type `<StringRefAddr>` with different types (`addrType`)) and an inner `<Contents>` tag that provides the content (in this case the cyphered password).

<br/>
<br/>
I don't know the exact details about the alghoritm that cyphers the passwords, and quite frankly I'm not that interested. One important note is that modern SQL Developer versions use a so called `v4` password masking alghoritm, that is based on a dynamic value tied to the machine the program is running on.
<br/>
Gosh!
<br/>
This means I cannot simply copy and paste my `connections.xml` file to another machine and get it magically working!
<br/>
<br/>
The key tied to the machine is stored into the file `product-preferences.xml` within the directory `~/.sqldeveloper/system4.1.5.21.78/o.sqldeveloper.12.2.0.21.78/` with, as usual, the numbering indicating the version of the SQL Developer.
<br/>
The `product-preferences.xml` file contains the dynamic key in the `db.system.id` attribute tag, for example:

<br/>
<br/>
```xml
<value n="db.system.id" v="aaaf8abb-a24b-4852-9d6d-ee11aa933383"/>
```
<br/>
<br/>

The value reminds me an *UUID*, and is used to cypher the passwords (I guess as a *salt*).


# Unencrypting stored passwords

Now the important part: how to decrypt passwords?
<br/>
I've found [this Python program](https://github.com/maaaaz/sqldeveloperpassworddecryptor){:target="_blank"} that accepts, as arguments, the encryptde password string and the dynamic key and provides the unencrypted password.
<br/>
The program is not able to read [yet](https://github.com/maaaaz/sqldeveloperpassworddecryptor/pull/1{:target="_blank"}) the `connections.xml` file, so you have to feed it manually with the password and the *salt*, but it has worked great an all my connections. As an example:


<br/>
<br/>
```shell
% pip install sqldeveloperpassworddecryptor

% /home/luca/.local/bin/sqldeveloperpassworddecryptor \
                    -p AbggYhjgbkbTEPqGvvTkmUU4I+99FuoM \
                    -d bb55ac37-a33b-4762-9a6c-bc012345abc3
sqldeveloperpassworddecryptor.py version 2.0

[+] encrypted password: AbggYhjgbkbTEPqGvvTkmUU4I+99FuoM
[+] db.system.id value: bb55ac37-a33b-4762-9a6c-bc012345abc3

[+] decrypted password: FLUCA1978
```
<br/>
<br/>

*NOTE: the above example has been re-edited to mask the actual hash and id value.*


# Conclusions

[sqldeveloperpassworddecryptor](https://github.com/maaaaz/sqldeveloperpassworddecryptor){:target="_blank"} can be very useful when trying to find out a password stored on a recovered machine or stuff like that.
<br/>
Personally I would prefer to have a tool to change all the passwords at once, since this is the activity I do the most, or even a way to update the passwords if the `db.system.id` changes.
