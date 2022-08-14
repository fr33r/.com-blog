+++
title = "Who's the Boss?"
date = "2022-08-13T08:12:39-07:00"
author = "fr33r"
cover = "img/the-office-managers.png"
tags = ["docker", "devops", "quickhit", "tutorial"]
keywords = ["docker", "manager"]
description = "Insights on Docker Swarm cluster managers and how you should deploy them."
showFullContent = false
readingTime = true
hideComments = false
+++

Every node in a Docker Swarm plays at least one of two particular roles: worker
and/or manager. When you create a new cluster, you start out by creating it with
a single node that is both a manager and a worker, and [specify an IP address
to advertise that manager to other managers][swarm-advertise] as you add them.

# Roles

So what are the responsibilities of these two roles anyway? How do you know
how many managers and workers to deploy in your swarm? Let's get into it!

## Managers

As you might imagine, [manager nodes][manager-nodes] in a Docker Swarm take on
additional cluster management tasks. Such tasks include:

- maintaining & distributing cluster state across manager nodes in the cluster
- scheduling services to various nodes in the swarm
- serving swarm mode HTTP API endpoints

## Workers

Intuitively, [worker nodes][worker-nodes] don't perform any of the management
tasks that the manager nodes do; all that stuff is above the pay grade.
Instead, worker nodes are solely responsible for executing service tasks.

## Promotion & Demotion

Roles for nodes can be changed. To [promote][swarm-promote] a worker node to
be a manager node, run the following:

{{< code language="bash" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
$ docker node promote <NODE NAME>
{{< /code >}}

And to [demote][swarm-demote] a node, run this command:

{{< code language="bash" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
$ docker node demote <NODE NAME>
{{< /code >}}

> _**NOTE**_: By default, all manager nodes are also worker nodes. Assuming you
have a cluster that has more than one node, you can prevent any one manager node
from taking on worker node responsibilities by moving it's availability status
to `Drain`.

# Deployment Strategies

Okay, so we've nailed down the roles that nodes can play within a swarm, but you
may be wondering:

> How do I choose which role(s) to assign to a node?

It's a great question, and one that can have pretty large impact.

## To Manage or Not to Manage

Just like any business, having too many or too little managers can be problematic.
When it comes to provisioning managers in a Docker Swarm, you want to strike the
right balance of **resiliency** and **performance**.

Let's touch first on resiliency.

### Resiliency

One of the primary reasons why we are using an orchestration engine like Docker
Swarm is so that we can have a degree of fault tolerance. If any particular
node in the swarm decides to take an "unnannounced vacation", we should be able
to schedule whatever tasks (containers that comprise a service) to a different,
active node.

We have to be particularly strategic to provide this fault tolerance when it comes
to managers. After all, they are responsible for taking on some pretty important
and necessary tasks. In order to best understand how many managers we should
deploy, we should quickly touch on the algorithm Docker Swarm uses to ensure
consistent state.

#### Raft Consensus Algorithm

Sounds fancy right?

The [Raft Consensus Algorithm][raft-algorithm] is essentially an algorithm that
Docker Swarm uses to make sure each manager in the cluster is storing the same
consistent state. The most important thing to note in the context of this discussion
is that manager nodes need to agree on what the state is, and that "agreement"
is achieved as a function of how many nodes share the same information. If the
majority, also referred to as a "quorum", is achieved when new state is proposed,
then it will be distributed to all managers.

Approaching this mathematically, we can essentially use the following simple
equations to determine achievable fault tolerance and quorum:

```
given:
  N = the number of manager nodes in the cluster.

then:
  number of manager nodes that can be down = (N-1)/2
  number of manager nodes to achieve quorum = (N/2)+1
```

> **_EXAMPLE:_** Assuming we have `3` manager nodes (`N = 3`) in a cluster of `5`
total nodes, we would be able to support `1` manager node being down at any
moment in time (`(3-1)/2 = 1`). The number of manager nodes to reach quorum
would be `2` (`(3/2)+1 = 2`).

Ultimately, the number of managers you deploy in your cluster depends somewhat
on the total number of nodes in your cluster. It is recommended you have at least
`3` manager nodes, as such a configuration allows for one manager node to be down.

### Performance

As with much in software design, the resiliency comes at a cost. Since one element
of increasing the resiliency involves introducing redundency, more work is required
to spread information as we introduce more managers. Adding more manager nodes,
means more managers are needed to meet quorum.

When it comes to Docker Swarm, it's generally [not recommended to have any more
than `7` manager nodes][manager-nodes]. Such a configuration would allow `3`
managers to be simultaneously down at any moment and require `4` nodes to reach
quorum.

### Unreachable Quorum

In the event that the number of manager nodes simultaneously down exceeds the
allowable fault tolerance, the state of the cluster is not allowed to change.
New tasks cannot be scheduled, and the existing tasks cannot be rebalanced to
other nodes if required. The following error will be returned when a management
operation is attempted:

```
Error response from daemon: rpc error: code = 4 desc = context deadline exceeded
```

The ideal way to recover here is to get manager nodes back online. If for some
reason the managers can't be brought back online, a new cluster will need to
be provisioned. This can be performed using the following command:

{{< code language="bash" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
$ docker swarm init --force-new-cluster --advertise-addr <IP>:2377
{{< /code >}}

Make sure to replace `<IP>` with the IP address or DNS resolvable name of the
host you desire to be the initial manager of this new cluster.

#### Workers Get Shit Done

Compared to manager nodes, worker nodes are far simpler. Since they are
designated for executing the tasks for the swarm, the main deployment concerns
are simply ensuring that there are enough workers to balance the load.

To join an existing swarm cluster as a worker node, run the following commands
to retrieve the worker join token:

{{< code language="bash" id="3" expand="Show" collapse="Hide" isCollapsed="false" >}}
$ docker swarm join-token worker
{{< /code >}}

Assuming the above command provided this output:

```
docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377
```

The final step would be to run that command:

{{< code language="bash" id="4" expand="Show" collapse="Hide" isCollapsed="false" >}}
$ docker swarm join \
  --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377
{{< /code >}}

> _**NOTE**_: When adding workers to accommodate for expected or current
workloads, make sure to take the various resource related requirements that
may be specified for a particular service. Remember that workers that
don't meet those specified requirements will not be scheduled to execute
those tasks.

[swarm-advertise]: https://docs.docker.com/engine/swarm/admin_guide/#configure-the-manager-to-advertise-on-a-static-ip-address
[manager-nodes]: https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/#manager-nodes
[worker-nodes]: https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/#worker-nodes
[raft-algorithm]: http://thesecretlivesofdata.com/raft/
[swarm-promote]: https://docs.docker.com/engine/reference/commandline/node_promote/
[swarm-demote]: https://docs.docker.com/engine/reference/commandline/node_demote/
