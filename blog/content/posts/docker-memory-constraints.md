+++
title = "If I Only Had More Memory"
date = "2022-08-12T14:01:08-07:00"
publishDate = "2024-08-31T21:22:01-07:00"
author = "fr33r"
cover = ""
tags = ["docker", "swarm", "devops"]
keywords = ["docker", "swarm", "memory", "constraint"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
draft = true
+++

When deploying a service to a cluster of hosts, it's important to ensure that
the host ultimately chosen can accommodate for the resource requirements of
that service. Without first confirming that the host can provide what the
service needs, it's possible that the service will consume all available
resources and result in degradation of the host itself.

# Memory Constraints

One such resource to keep tabs on is memory (RAM). Each process that is ran
within a particular container will consume some amount of memory while it is
running.

By default, Docker does not apply resource constraints to containers that
ultimately comprise a service. That said

# How 

# Reaching the Limit

So what happens once the limit is reach



what are docker memory constraints?
why should you use them?
how do i determine the memory limit or reservation?```
what is OOM and how to detect it?
what's the deal with swap? should you disable it?
link to articles that could be helpful:

- https://github.com/kubernetes/kubernetes/issues/53533
- https://haydenjames.io/linux-performance-almost-always-add-swap-space/
- https://docs.docker.com/config/containers/resource_constraints/#prevent-a-container-from-using-swap
- https://docs.docker.com/compose/compose-file/deploy/#resources
- https://www.baeldung.com/ops/docker-memory-limit#:~:text=Versions%203%20and%20Newer%20With,and%20128%20megabytes%20of%20memory.&text=To%20take%20advantage%20of%20the,use%20the%20docker%20stack%20command.
- https://docs.docker.com/engine/install/linux-postinstall/
- https://forums.docker.com/t/docker-swarm-not-respecting-memory-limit/59519
- https://semaphoreci.com/community/tutorials/scheduling-services-on-a-docker-swarm-mode-cluster


vader@devastator:~ $ sudo docker container inspect 6fd463496083 --format '{{.State.OOMKilled}}'
false
