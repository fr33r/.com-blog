+++
title = "Configuration NFS with Firewalls"
date = "2023-03-05T13:02:04-08:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
draft = true
+++

Storing and retrieving files from a remote location is a common need for
distributed cluster environments. Given any one service or application can
be ran from a pool of different hosts, those services may need to potentialy share
configuration, data, and other files between them.

For Unix based hosts, one such option for this is known as
[Network File System](https://en.wikipedia.org/wiki/Network_File_System),
or NFS for short.

## What is NFS?

NFS is a distributed file system protocol. It essentially emulates remote file
access as if that remote location was local to the client. It was originally
developed by Sun Microsystems all the way back in 1984, and has had several
versions and variations over the years.

### Versions

Despite there being four official versions, I'll concentrate on the latest two:
NFSv3 and NFSv4. These are the two versions you are likely to interact with in
todays world, and versions older than NFSv3 have several limitations that no 
longer make them viable (storage size limitations, performance limitations, etc.).

#### NFSv3

NFS really comprises of several different organized components:

Network Status Monitor (NSM) Protocol

[rpc.nfsd](https://linux.die.net/man/8/rpc.nfsd)

The NFS server process. By default, it runs on port `2049`.

[rpc.statd](https://linux.die.net/man/8/rpc.statd)

A client and server-side daemon responsible for listening for reboot notifications
from other hosts, while also doing the flip-side by managing the list of hosts to
be notified if the local system reboots.

`rpc.statd` is what the NFS Lock Manager (NLM) communicates with to help maintain
proper file locking state when NFS connected hosts reboot.

By default, it runs on a random ephemeral port for each socket. This
daemon is only needed for NFSv3 and NFSv2.

[rpcinfo](https://linux.die.net/man/8/rpcinfo)

A command line utility used to discover what various RPC services have been registered
with `rpcbind` on a host. For use cases of working NFS, this utility can be used
to view what NFS-related services are running and what ports they are configured on.

[rpc.mountd](https://linux.die.net/man/8/mountd)

The NFS server-side daemon responsible for receiving and managing mount requests
to NFS exports. By default, it runs on a random ephemeral port for each socket. This
daemon is only needed for NFSv3 and NFSv2.

#### NFSv4

## Running a Firewall with NFS

Given the default behavior of the various components of NFSv3 using random
ephemeral ports, some configuration changes are needed to instead utilize static
ports. 

>

###	NFS v3

### NFS v4
