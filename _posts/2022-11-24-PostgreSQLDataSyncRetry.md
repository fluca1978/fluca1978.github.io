---
layout: post
title:  "PostgreSQL scary settings: data_sync_retry"
author: Luca Ferrari
tags:
- postgresql
- planet-postgresql-org
permalink: /:year/:month/:day/:title.html
---
A look at how this setting works.

# PostgreSQL scary settings: data_sync_retry

Is this a scary setting? Well, of course no!
<br/>
However, it is a setting that you should not touch unless you are really, really aware of what you are doing.
<br/>

[`data_sync_retry`](https://www.postgresql.org/docs/current/runtime-config-error-handling.html){:target="_blank"}  is a setting that instrument the cluster to **retry after an `fsync` failure** related to data pages. What does that mean?
As we all know, PostgreSQL has to flush data, sooner or later, from memory to the data files, and this happens with `fsync(2)`, an operating system call that forces the data to be flushed from memory to the filesystem layer and, hopefully, to the disk into the data files.
<br/>
Clearly, PostgreSQL cannot allow any data loss, even of one bit, and therefore the system takes great care about what happens when flushing data.
In normal circumstances, if `fsync(2)` fails, data has not been written to disk and therefore there is nothing PostgreSQL can do about it.
Since this is a *huge problem*,  **PostgreSQL issues a `PANIC`** and crashes.
That's not so good, but it is safe after all, since it means we are going to recover from the WALs and therefore we are not going to loose any data.
<br/>
In the case we force a *retry*, by means of setting `data_sync_retry` to `on`, PostgreSQL will not crash, so that later on the flush-to-disk could be retried (hence the name of this setting).
<br/>
<br/>
So, why you should not enable such a great setting that promises to you to avoid crash even when `fsync(2)` fails?
<br/>
The problem is that, after an `fsync(2)` failure, the kernel pacge cache status could be unknown. This means that the page, that was moved to the page cache (i.e., filesystem layer) to be flushed, could have been removed from the kernel cache even if did not hit the disc (because `fsync(2)` was responsible for this, and it failed). In such situation, PostgreSQL could have discarded the dirty page from the shared buffers, the kernel could have thrown away the page, and the page did not hit the disc. At this point, a retry happens, and the operating system reports a success, making things even worst! **And here's where the data loss happens!**

## How does PostgreSQL handles the above?

If you take a look at the code, in particular at the function [`data_sync_elevel`](https://github.com/postgres/postgres/blob/master/src/backend/storage/file/fd.c#L3736){:target="_blank"}:

<br/>
<br/>
```c
int
data_sync_elevel(int elevel)
{
	return data_sync_retry ? elevel : PANIC;
}

```
<br/>
<br/>

you will see that, unless the `data_sync_retry` option is enabled, the system returns a `PANIC`.
Such function is used whenever a call to `fsync(2)` stuff happens, [for example](https://github.com/postgres/postgres/blob/master/src/backend/storage/file/fd.c#L507){:target="_blank"} :

<br/>
<br/>
```c
rc = sync_file_range(fd, offset, nbytes,
					 SYNC_FILE_RANGE_WRITE);
if (rc != 0)
{
	int			elevel;

    ...
    elevel = data_sync_elevel(WARNING);

	ereport(elevel,
			(errcode_for_file_access(),
			 errmsg("could not flush dirty data: %m")));
}
```
<br/>
<br/>

For short: when there is an error in data syncing, PostgreSQL tries to understand at which level it must emit the problem. Here the system decides, thru `data_sync_level` to choose for `WARNING` (in the case there will be another try) or the `PANIC` (the default).


# Conclusions

Similarly to what happens for `fsync`, that you should never change and keep always set to `on`, the `data_sync_retry` setting must be never touched and should always retain its default value `off`. Turning `on` this parameter could result in data loss, and surely does not provide you any performance benefit.
<br/>
An interesting thread to [read about](https://www.postgresql.org/message-id/957805.1668461398%40sss.pgh.pa.us){:target="_blank"} that provides interesting concepts and explaination.
