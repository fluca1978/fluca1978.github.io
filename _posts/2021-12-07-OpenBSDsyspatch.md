---
layout: post
title:  "OpenBSD syspatch"
author: Luca Ferrari
tags:
- openbsd
permalink: /:year/:month/:day/:title.html
---
syspatch is a very good tool to apply patches on OpenBSD

# OpenBSD syspatch

OpenBSD issues patches when bugs or vulnerabilities are discovered and, consequently, fixed.
<br/>
The tool that allows you to apply patches is `syspatch`, and it is very simple to use. First of all, when booting an OpenBSD machine, `syspatch` will search for new patches and display a warning message on the boot console.
<br/>
For a long running system, you can always run `syspatch` to get the list of applicable pathces, such as for a 7.0 machine:


<br/>
<br/>
```shell
% doas syspatch -c
001_nsd
002_bpf
003_uipc
004_rpki
005_unpcon
006_x509

```
<br/>
<br/>

The `-c` option lists all new patches; the meaning for the `-c` is that this has been thinked for a scheduled run of `syspatch` by means of `cron` (hence, the mnemonic `c`).
<br/>
In order to apply the patches, it does suffice to run the command without any specific option:

<br/>
<br/>
```shell
% doas syspatch
Get/Verify syspatch70-001_nsd.tgz 100% |***********************************************************************************|   760 KB    00:00    
Installing patch 001_nsd
Get/Verify syspatch70-002_bpf.tgz 100% |***********************************************************************************|   106 KB    00:00    
Installing patch 002_bpf
Get/Verify syspatch70-003_uipc.tgz 100% |**********************************************************************************| 91867       00:00    
Installing patch 003_uipc
Get/Verify syspatch70-004_rpki.tgz 100% |**********************************************************************************|   168 KB    00:00    
Installing patch 004_rpki
Get/Verify syspatch70-005_unpcon.tgz 100% |********************************************************************************| 91953       00:00    
Installing patch 005_unpcon
Get/Verify syspatch70-006_x509.tgz 100% |**********************************************************************************| 17614 KB    00:02    
Installing patch 006_x509
Relinking to create unique kernel...done; reboot to load the new kernel
Errata can be reviewed under /var/syspatch
```
<br/>
<br/>

Once the patching has completed, **you need to reboot**!

## Rollbacks

`syspatch` stores the files every single patch is going to change into a folder named after the patch, under `/var/syspatch`, in an archived named `rollback.tgz`. This allows `syspatch` to *rollback* a patch with the `-r` option. Let's inspect the patches applied previously:


<br/>
<br/>
```shell
% ls -s1h /var/syspatch/*
/var/syspatch/70-001_nsd:
   4 001_nsd.patch.sig
1568 rollback.tgz

/var/syspatch/70-002_bpf:
  4 002_bpf.patch.sig
108 rollback.tgz

/var/syspatch/70-003_uipc:
 4 003_uipc.patch.sig
92 rollback.tgz

/var/syspatch/70-004_rpki:
372 004_rpki.patch.sig
232 rollback.tgz

/var/syspatch/70-005_unpcon:
 4 005_unpcon.patch.sig
92 rollback.tgz

/var/syspatch/70-006_x509:
   16 006_x509.patch.sig
35264 rollback.tgz

```
<br/>
<br/>

In order to rollback a single change, use the `-r` flag:

<br/>
<br/>
```shell
% doas syspatch -c
006_x509
```
<br/>
<br/>

The `-r` flag reverts the most recent change, so you can iterate on `-r` to go back in history one patch at a time. However, if you want to revert *all* the changes, use `-R`.
<br/>
Sometimes, things can *go wrong*:

<br/>
<br/>
```shell
% doas syspatch -r
Reverting patch 005_unpcon
Relinking to create unique kernel... failed!
!!! "/usr/libexec/reorder_kernel" must be run manually to install the new kernel
```
<br/>
<br/>


## The problem with `reorder_kernel`

Sometimes it could happen a weird, at least to me, error message:

<br/>
<br/>
```shell
% doas syspatch
syspatch: cannot apply patches while reorder_kernel is running
```
<br/>
<br/>

As far as I understand, that means that the program `reorder_kernel` did not complete succesfully, and in other words a possible `syspatch` failed. In order to fix that, and allow `syspatch` to work again, you need to:

<br/>
<br/>
```shell
# sha256 /bsd > /var/db/kernel.SHA256
# /usr/libexec/reorder_kernel
```
<br/>
<br/>

The first step is optional, but since `reorder_kernel` could take a long time, I suggest you to do it in order to avoid having to re-run `reorder_kernel`. A reboot, in this case, should not be needed.

