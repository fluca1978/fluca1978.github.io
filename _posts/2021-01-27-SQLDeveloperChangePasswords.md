---
layout: post
title:  "Oracle SQL Developer: Changing Connection Passwords Massively"
author: Luca Ferrari
tags:
- emacs
- raku
permalink: /:year/:month/:day/:title.html
---
A possible way to change all the passwords at once!

# Oracle SQL Developer: Changing Connection Passwords Massively

I tend to use Oracle SQL Developer as my main fornt-end for Oracle databases, and it works quite well after all.
<br/>
However, by time to time, I'm forced to change my personal passwords by enterprise policies, and this has always been a tedious task because I have to manually edit the *connection properties* for every single connection and update the old password with the new one.
<br/>
<br/>
And then, this morning, after a strong coffee, I was enlightned.
I do use *proxy* accounts to pretty much all my databases, ops, *schemas* as they are called in Oracle terminology. This means my connection username is something like `luca[db]` or `luca[db2]`: the square brackets include the schema name, and the prefix is my own username· This is really useful because it allows my user to connect **with the very same credentials** to different schemas without having to know the schema password.
<br/>
On the other hand, this means that SQL Developer **stores the same password for all the *proxy based* connections** I've defined. Well, not really true: SQL Developer stores a cryptogrphic hash of the same password for all the above connections.
<br/>
Do you see when I'm going? **If I can tell the new cryptographic hash for a new password, I can quickly perform a *substitute-all* of all the occurencies in the connection configuration file**.
<br/>
Unluckily, I'm not aware of a way of getting a new encrypted hash starting from a password (while I'm aware of how to decrypt an existing hash). So here follows what I did to keep my idea as *massive* as possible.

## First Step: Analyze the Connection Properties

SQL Developer stores the connection properties into an XML file named `connections.xml` that is, in turn, within the user home directory under the `.sqldeveloper` path, in a folder named `system<something>` and a subfolder named after the `connections` string. On my system it is something as:

<br/>
<br/>
```
$HOME/.sqldeveloper/system4.1.5.21.78/o.jdeveloper.db.connection.12.2.1.0.42.151001.541/connections.xml
```
<br/>
<br/>

In the above `connections.xml` file, there are different sections, one per connection, each with a *password* section:


<br/>
<br/>
```xml
 <StringRefAddr addrType="password">
    <Contents>==passwordHashHere==</Contents>
 </StringRefAddr>
```
<br/>
<br/>

The `Contents` is the password hash for that connection.
<br/>
The hash is computed using a key that is based on the current host, so **you cannot use the same password hash across different machines**.
<br/>
But guess what: who cares!
<br/>
Just esnure that all the password hash for the connections you are going to edit are the same string, that means the same password.

## Second Step: Obtain a New Password Hash

Let SQL Developer do the heavy job: launch it and change *only one* connection among yours with the new password. Then save the connection and close the SQL Developer.


## Last Step: Do the Substitution

Now open the `connections.xml` file with you default editor (and that should be Emacs!).
Find out the connection you have already updated with the new password, grab the password hash and perform a *replace-all* of the old password hash with the new one.
<br/>
Save the file and launch SQL Developer again, and you are done!
