+++
title = "Nfs and Firewalls"
date = "2023-03-12T11:13:44-07:00"
author = "fr33r"
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

Network File System (NFS) and the various components that comprise it are,
by default, not condusive to be setup with firewalls attempting to restrict
traffic using ports. This is primarily because several components of NFSv3
utilize random port assignments.

Not to worry, I've got you covered.

### What We Will Talk Through

- NFS versions and how to determine which are running on your machine.
- The components that comprise NFSv3 and NFSv4.
- Commands and utilities to observe NFS configuration.
- How to use static port assignments for each component of NFSv3.

### What We will NOT Talk Through

- How to install or use NFS.
- How to install a firewall and associated tooling (such as `ufw`).
- Anything about NFSv2 and below.

> NOTE: There are tons of awesome tutorials for setting up NFS servers and clients
for nearly any OS imaginable. You shouldn't have any trouble getting those up
and running for your needs. Same goes for various firewalls and related tools.
Lastly, we won't chat about NFSv2 because it's old, not really viable in today's
world, and on some operating systems, even disabled by default.

#### Which Versions are on My Machine?

When folks refer to NFS, it's really a reference to all of the components that
make up NFS as well as it's various versions.

NFS installations usually have more than one NFS version running simultaneously.
If you are using a Linux distro, you can check out which versions are enabled
and disabled like this:

```bash
> sudo cat /proc/fs/nfsd/versions
-2 +3 +4 +4.1 +4.2
```

As you can see here, on my machine NFSv2 is disabled, and NFSv3 and NFSv4 are enabled.
This is typically the default configuration after installation.

#### Components & Utilities

Remember when I said NFS is made up of a bunch of different components? NFSv3
in particular comprises of an orchestration of several daemons and other programs.
Let's walk through those and some utilities that will help us navigate them.

##### [`rpc.nfsd`](https://linux.die.net/man/8/rpc.nfsd)

The NFS server process. By default, it runs on port `2049`.

##### [`rpc.statd`](https://linux.die.net/man/8/rpc.statd)

A client and server-side daemon responsible for listening for reboot notifications
from other hosts, while also doing the flip-side by managing the list of hosts to
be notified if the local system reboots.

`rpc.statd` is what the NFS Lock Manager (NLM) communicates with to help maintain
proper file locking state when NFS connected hosts reboot.

By default, it runs on a random ephemeral port for each socket. This
daemon is only needed for NFSv3 and NFSv2.

##### [`rpc.mountd`](https://linux.die.net/man/8/mountd)

The NFS server-side daemon responsible for receiving and managing mount requests
to NFS exports. By default, it runs on a random ephemeral port for each socket. This
daemon is only needed for NFSv3 and NFSv2.

##### [`rpcinfo`](https://linux.die.net/man/8/rpcinfo)

A command line utility used to discover what various RPC services have been registered
with `rpcbind` on a host. For use cases of working NFS, this utility can be used
to view what NFS-related services are running and what ports they are configured on.

#### Configuring Static Ports

In order to successfully use a firewall with NFSv3 and above, we need to make
sure those ports don't change! The primary way we do that on Linux distributions
is by editing the `/etc/sysconfig/nfs` file.

> NOTE: It's totally possible this file doesn't exist on your machine. Simply create
it before continuing on.

Open the file with your editor of choice, and specify the following configuration:

```
MOUNTD_PORT=<port number>
STATD_PORT=<port number>
LOCKD_TCPPORT=<port number>
LOCKD_UDPPORT=<port number>
````

Make sure to replace each instance of `<port_number>` with an unused port. You can
check if a desired port number is already taken like so:

{{< code language="bash" id="" expand="Show" collapse="Hide" isCollapsed="false" >}}
> lsof -i :<port number>
{{< /code >}}

Since the default port of the NFS server process is already a static port (`2049`),
I chose to sequence the remaining port numbers to make them easier to remember
and identify:

{{< code language="bash" id="" expand="Show" collapse="Hide" isCollapsed="false" >}}
MOUNTD_PORT=2050
STATD_PORT=2051
LOCKD_TCPPORT=2052
LOCKD_UDPPORT=2053
{{< /code >}}

Once you have made your changes to `/etc/sysconfig/nfs`, we will need to restart
NFS for them to take affect:

{{< code language="bash" id="" expand="Show" collapse="Hide" isCollapsed="false" >}}
sudo systemctl restart nfs-kernel-server
{{< /code >}}

Lastly, we can verify that these ports are now being leveraged by using `rpcinfo`:

{{< code language="bash" id="" expand="Show" collapse="Hide" isCollapsed="false" >}}
> rpcinfo -p
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  46174  status
    100024    1   tcp  53673  status
    100005    1   udp   2050  mountd
    100005    1   tcp   2050  mountd
    100005    2   udp   2050  mountd
    100005    2   tcp   2050  mountd
    100005    3   udp   2050  mountd
    100005    3   tcp   2050  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049
    100003    3   udp   2049  nfs
    100227    3   udp   2049
    100021    1   udp   2051  nlockmgr
    100021    3   udp   2051  nlockmgr
    100021    4   udp   2051  nlockmgr
    100021    1   tcp   2051  nlockmgr
    100021    3   tcp   2051  nlockmgr
    100021    4   tcp   2051  nlockmgr
{{< /code >}}

#### Conclusion

### Resources

- [](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/s2-nfs-nfs-firewall-config)
- [](https://wiki.debian.org/SecuringNFS)
- [](https://www.suse.com/support/kb/doc/?id=000016649)

