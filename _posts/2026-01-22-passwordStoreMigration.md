---
layout: post
title:  "Migrating a password store to another machine"
author: Luca Ferrari
tags:
- freebsd
- linux
permalink: /:year/:month/:day/:title.html
---
A quick guide on how to move (and use) a password store.

# Migrating a password store to another machine

I use the great **[pass](https://www.passwordstore.org/){:target="_blank"}** as a password store on my Unix-like machines.
It is fast, simple enough, can work on a terminal, can copy the password on the clipboard, and so on.

Sometime I need to move the password store, organized into the **`~/.password-store`**, from a machine to another to get my password at my fingertips.

The following are the steps to achieve the aim.

## Export your GnuPG key

`pass` uses `gpg` to store encrypted password files in the store, so the first step is to export the key:

<br/>
<br/>
```shell
% gpg --export-secret-keys user@foo.bar > key.asc

```
<br/>
<br/>

In the above, substitute the `user@foo.bar` with your own key id. You can get the list of the keys and identifiers by running `gpgp -k`.

Then, copy somehow (e.g., via `scp`) the key to the target machine.


## Import the GnuPG key

Once you have the key, you can import the key on the target machine:

<br/>
<br/>
```shell
% gpg --import key.asc
...

gpg: key AC68C777777777: secret key imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1

```
<br/>
<br/>


## Copy the password store

You need to copy and keep in sync the password store.
The method is up to you, you can use `git` or any other tool to synchronize the password store.


## Test `pass` is working

On the target machine, either run `pass` to see the tree view of the store or `pass ls`, then try to get out a password to verify everything is working.


# Conclusions

`pass` is a very powerful tool, and it can be used to quickly make your own password storage grow and be shared among different machines.
