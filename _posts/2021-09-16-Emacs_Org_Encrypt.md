---
layout: post
title:  "Encrypting the content of Emacs Org files" 
author: Luca Ferrari
tags:
- emacs
- org-mode
permalink: /:year/:month/:day/:title.html
---
A quick look at how to encrypt sections within an Org file.

# Encrypting the content of Emacs Org files

Org mode is a super tool for managing *your life in plain text*.
<br/>
However it has a problem: the content is, well, *plain text*. This can quickly become an obstacle when you are going to deal with distributing your org files across several machines, e.g., by means of a common repository.
<br/>
After all, we all have secrets!
<br/>
<br/>
However, Org mode has a builtin capability to encrypt sections of a tree so that you can still have your plain text files, but its content will not be shown to someone else.
<br/>
The idea is simple:
- you configure a particular *tag*  to add to an header;
- whenever the file is going to be saved, the sections with the heading containg the above tag will be encrypted.

<br/>
<br/>
**Danger Will Robinson!** 
<br/>
This is not a perfect tool, and there are a few different ways to let information to escape. For example, if you *cancel* the request for an encryption password, the file will be saved on disk as full plain text. Again, if you have auto-save enabled, the backup files will have no encryption at all.
<br/>
<br/>

## Configuring Emacs for (symmetric) Org Mode Encryption

The example shown here is taken from the [official manual page of `org-crypt`](https://orgmode.org/worg/org-tutorials/encrypting-files.html){:target="_blank"}.
<br/>
In your Emacs configuration file place the following:

<br/>
<br/>
```lisp
(require 'org-crypt)
(org-crypt-use-before-save-magic)
(setq org-tags-exclude-from-inheritance (quote ("crypt")))
(setq org-crypt-key nil)
```
<br/>
<br/>

The first line enables the `org-crypt` feature, the second line tells the encryption to activate whenever a file is saved. The third line defines the *tag* that will trigger the encryption, and the last line tells no particular encryption key will be used. When no key is specified, Emacs will prompt you for an encryption passphrase, that will be used for both encryption and decryption.

## Symmetric Encryption in Action

Let's start with a very simple Org file: I place a few sections and nest two sections (level two headings) that must be encrypted. I therefore place the `crypt` tag on the headings of the sub-sections I want to encrypt and then save the file.
All the sections with the `crypt` tag will be encrypted when you ask for saving the file. In fact, as soon as you hit `C-x C-s`, a dialog window pops up asking you for the passphrase (you have to enter it twice):

<br/><br/>
<img src="/images/posts/emacs/org_encrypt_1.png" />
<br/>

<br/>
**the content of the file changes: the sections with the `crypt` tag now show an encrypted bunch of text**:

<br/>
<br/>
<img src="/images/posts/emacs/org_encrypt_2.png" />
<br/>




<br/>
<img src="/images/posts/emacs/org_encrypt_3.png" />
<br/>


## Decrypting a Section

The special command `org-decrypt-entry` will decrypt the Org entry at the point, that is you can place the cursor wherever within the encrypted text and the section will be decrypted (of course, by means of the passphrase):

<br/>
<br/>
<img src="/images/posts/emacs/org_encrypt_4.png" />
<br/>
<br/>
<img src="/images/posts/emacs/org_encrypt_5.png" />
<br/>
<br/>
<br/>
There is also the command `org-decrypt-entries` that will automatically decrypt any section you have within the file.


# Conclusions

`org-crypt` is an easy embedded feature that allows you to encrypt sections of your Org files, and this is an enhancement that allows us to store such files on remote and publicly available locations without revealing confidential data.
<br/>
Of course this is not the only tool we have, but it does suffice in day-by-day usage and simple scenarios. More complex workflow could require a full file encryption by means of tools like `gpg` and alike.
<br/>
It is important to be aware that `org-crypt` does not provide a perfect workflow, and there could be some ways to obtain access to plain information.
