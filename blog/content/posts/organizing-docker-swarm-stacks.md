+++
title = "Writing Better Docker Swarm Stacks"
date = "2023-02-10T21:51:53-08:00"
author = "fr33r"
cover = "img/jon_blog_writing_better_stacks_cover.png"
tags = ["docker", "swarm", "devops", "tips"]
keywords = ["docker", "swarm", "stack", "tips", "devops", "portainer"]
description = "Tips and tricks for writing better Docker Swarm stack files."
showFullContent = false
readingTime = true
hideComments = false
+++

When I decided to stand up my home lab, I had little to no prior experience
with Docker Swarm or other supporting infrastructure (shared storage,
node provisioning, etc.) and applications (Portainer, NAS software, etc.). The process of
building out my cluster has been packed with learnings from both the good and
the bad experiences I've had along the way. Additionally, I've picked up some
patterns and practices that have helped me get to where everything is today.

One area specifically worth sharing is regarding Docker Compose (stack) files.
This post walks through the most critical bits that inform how I organize
and write my stack files.

### Multiple Stack Files

It can be tempting to put all of your service definitions in a single stack file at first.
After all, it means you only have one file to worry about; everything is in a single place.
Also, you don't have to worry about setting up a directory structure to organize - keeping things
tidy takes effort!

However, I encourage you to fight the urge. There are some really tangible benefits
to breaking your stack files down by application:

- Each stack file becomes self contained. The services defined in the stack file
are specific to a partular application you are spinning up. All of the volumes,
networks, etc. are all specific to the service definitions in the file; you're no
longer trying to determine how everything ties together.
- Separating the files means you can deploy a finite set of services at a time
instead of all of them. This helps make your deployments more manageable and
ensures that you aren't going to have unintended side-effects with other
services.
- There is less likelihood that you will accidentally reference an element
unintentionally or have a naming clash. If performing variable substitution, you
won't have to worry about inadvertently giving two variables the same name.
- Each stack file is simply easier to read. Any one application usually doesn't have
more than a few services (web server, database, cache, etc.) which is easier to
navigate than a single file with dozens of services.
- Tools such as Portainer provide an interface for managing stacks separate from
the services they define. With the files broken out by application, you can
manage each stack independently using the UI.

As with anything, there are some drawbacks. When splitting out files, you
likely will have to repeat particular dependency definitions that various services of
yours utilize. For example, you will need to replicate a network definition in
each stack file that defines services that should be a part of that network. In
the event you need to change one of those dependencies, you will need to visit multiple
stack files to apply that change, which is a measurable amount of more work.

For me, the benefits outweighed drawbacks.

### Service Naming Conventions

Every stack file defines a list of services. Often times your stack will
define a single service. However, some applications that you wish to deploy
could be more complex, including a web server, database, cache, job queue,
and more.

When it comes to naming these services, I've found that naming them after the
role they play as the most effective. More specifically, I use the following
within my stacks:

- `app`
- `db`
- `cache`
- `queue`
- etc.

There are many reasons why this strategy has worked for me:

- The service names aren't coupled to any particular implementation, which means
that switching them to something else ( let's say, switching from `Redis` to `Memcache` )
won't ripple up to the service names.
- It's easier to navigate your services. All your database services are end in`db`.
All of your caches are end in `cache`.
- Since service names are prefixed by the name of the stack they belong to, this
strategy results in service names that avoid commonly seen mistakes, such
as repetitive words within the name.

### Pinning to Specific Image Versions

It's easy to leave off a specific image version or to specify `latest`.
However, this can really burn you, and typically at the worst time. Instead,
you should explicitly specify an image version (tag).

Why? Well the are several reasons:

- In a cluster environment, images are pulled down on each node when the service is scheduled to run on them.
If at any time you add in additional nodes, or deploy to a node you hadn't previously, it is possible
that the images on each node for the service aren't the same. After all, `latest`
is a moving target, and it's not trivial to determine when new releases are cut.
- In situations where your node(s) experience a more catestrophic event (hardware
failure, botched OS upgrade, etc.) restoring the data and configuration for the
service(s) is half the battle, but running a version of the service compatible
with that configuration and data is the other half. Remember that the data and configuration that one
version of your service utilizes could be completely different from the current
(latest), and the last thing you want to do is spend time triaging errors that
are only happening because you're running an incompatible version.
- It becomes much more apparent what version you are running for your services,
which makes it easier to search for bugs, documentation, and other artifacts
online.
- Upgrading your services becomes more intentional. You specify the next version you
want to upgrade to explicitly. This approach can help you avoid upgrading to
versions known to be less stable or haven't been well tested yet.

### Source Control for Stack Files

Your stack files are essentially the source of truth for your cluster setup;
you should manage and track changes to them so you keep your sanity! So much tweaking,
changing, improving, and learning happens when you work in a cluster environment.
It's going to be near impossible to keep all of those learnings in recent memory.
Using source control is an incredibly helpful tool to track all of that context
for future you, while also giving you the safety net of being able to roll back
your changes gracefully.

An awesome side-effect of pushing your stack files to a remote source control
such as GitHub is applications like Portainer can integrate directly with
your stack repositories. This means you can make changes to your stacks and
push them to GitHub, and then you can use Portainer to deploy those changes.

### Utilize Variable Substitution for Image Tag

Docker compose (stack) files support [variable substitution][docker-compose-var-sub], which means that
various elements of the stack can be dynamically evaluated based on environment
variables. This is even easier if you use a container management platform such
as Portainer, as it allows you declare the environment variables when you
create a stack directly in the UI.

There are various use cases one could use variable substitution for, but one
that I strongly recommend is using them to specify the image version (tag). This way
you can manage upgrades for your services by simply changing the environment
variable and redeploying the stack. If you manage your stacks using source countrol
this means you can upgrade the image without any need to change the stack file.

### Leverage Docker Secrets & Config

Many services store their configuration using one or more static configuration files.
For example an `nginx` proxy may be configured such that each domain that it
handles has a separate `*.conf` file. For services that are primarily driven through
updates of static configuration files, [Docker Configs][docker-configs] is a perfect fit. Using
Docker Configs in a swarm environment means you can edit these configuration
files in a streamlined and centralized way, can version that configuration,
and no longer need to set up the necessary infrastructure to ensure that
configuration is shared across all nodes that the service runs on.

An additional common use case for services is specifying credentials. Many Dockerized
applications provide a way to specify credentials via environment variables. Some
services also rely on TLS/SSL configuration. These use cases are where [Docker
Secrets][docker-secrets] shines. Instead of passing in credentials using variable subsitution,
or instead of creating infrastructure to share certificate information across
several nodes, you can use Docker Secrets similarly to Docker Configs but for
sensitive "secret" data.

I'm just scratching the surface with these. Expect a future blog post on how
I use both of these in my cluster setup!

[docker-compose-var-sub]: https://docs.docker.com.xy2401.com/compose/environment-variables
[docker-configs]: https://docs.docker.com/engine/swarm/configs/
[docker-secrets]: https://docs.docker.com/engine/swarm/secrets/
