+++
title = "Tale of the Unknown GitHub Secrets"
date = "2022-08-10T20:43:31-07:00"
author = "fr33r"
cover = "img/github-secrets.jpeg"
tags = ["nerdy", "fix", "devops"]
keywords = ["github", "actions", "docker", "secrets", "environments"]
description = "Github environments & secrets."
showFullContent = false
readingTime = true
hideComments = false
+++

When building out the deployment pipeline for this blog, I had my first run-in
with [Github Actions](https://github.com/features/actions). In particular, my goal was to automatically generate a
new Docker image when I push changes to the GitHub repository containing
the source files, and push that Docker image to my corresponding DockerHub repository.

# What secrets?

In order to achieve this, I reached for [`docker/login-action`](https://github.com/marketplace/actions/docker-login). This straightforward
action essentially takes the credentials provided to it and logs into DockerHub
using those credentials.

I went ahead and bootstrapped a [workflow configuration](https://github.com/fr33r/.com-blog/blob/main/.github/workflows/docker.yml) based on the example
provided in the [documentation for the DockerHub registry](https://github.com/marketplace/actions/docker-login), and created the
`DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD` secrets in GitHub.

Then it was time to test. I went ahead and pushed some changes to trigger the
workflow, and then...

{{< code language="output" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
Run docker/login-action@v2
  with:
    ecr: auto
    logout: true
Error: Username and password required
{{< /code >}}

Hmm, strange.

I know I created the secrets and [specified them in the workflow configuration](https://github.com/fr33r/.com-blog/blob/352aa1c2bc88345e05efe3f9174643f0789e2abf/.github/workflows/docker.yml#L25-L26).
Why didn't they come through? Is this a generic error message, or does it truly
get displayed when the credentials are missing? Did I forget to add values for the
secrets? Do the keys for the secrets line up in the workflow configuration?

To help me find some answers, I reran the failed workflow with debug logs enabled:

{{< code language="output" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
##[debug]Evaluating condition for step: 'Login to DockerHub'
##[debug]Evaluating: success()
##[debug]Evaluating success:
##[debug]=> true
##[debug]Result: true
##[debug]Starting: Login to DockerHub
##[debug]Register post job cleanup for action: docker/login-action@v2
##[debug]Loading inputs
##[debug]Evaluating: secrets.DOCKERHUB_USERNAME
##[debug]Evaluating Index:
##[debug]..Evaluating secrets:
##[debug]..=> Object
##[debug]..Evaluating String:
##[debug]..=> 'DOCKERHUB_USERNAME'
##[debug]=> null
##[debug]Result: null
##[debug]Evaluating: secrets.DOCKERHUB_TOKEN
##[debug]Evaluating Index:
##[debug]..Evaluating secrets:
##[debug]..=> Object
##[debug]..Evaluating String:
##[debug]..=> 'DOCKERHUB_TOKEN'
##[debug]=> null
##[debug]Result: null
##[debug]Loading env
Run docker/login-action@v2
  with:
    ecr: auto
    logout: true
::save-state name=isPost::true
##[debug]Save intra-action state isPost = true
::save-state name=registry::
##[debug]Save intra-action state registry = 
::save-state name=logout::true
##[debug]Save intra-action state logout = true
Error: Username and password required
##[debug]Node Action run completed with exit code 1
##[debug]Finishing: Login to DockerHub
{{< /code >}}

Okay. So the workfow is attempting to resolve the secrets, but it seems it is printing
out `null` for both the `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`. I remember seeing
[something](https://docs.github.com/en/actions/security-guides/encrypted-secrets#accessing-your-secrets) about values of secrets being redacted from action output, but does that mean
that it will print `null`? That can't be right...

# Root Cause + Fix

After spending far too much time turning over these stones, I finally discovered the
concept I was lacking knowledge on: GitHub environments.

Being totally new to setting up secrets in GitHub actions, I had missed the [documentation](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#using-an-environment)
that described how workflows interacted with GitHub environments. Equipped with this ignorance,
I proceeded to create the GitHub secrets as environment secrets, not repository
secrets. Combine this naiveness with the documentation provided for
`docker/login-action`, which outlines examples that implicitly assumes
repository level secrets are used, and the end result is that the workflow
configuration I created couldn't access the secrets I had created.

There were two solutions for here:

- make my workflow configuration environment aware, or
- remove the existing secrets I had created and add them as repository level secrets.

Given I had created the environment without much intention, and because I currently
don't have much of a use case for supporting multiple environments for this project,
I chose to go with the latter approach. Once the credentials were moved (and the old
ones removed), it worked like a charm!

> Had I chosen to go with the first option, I would've added an `environment`
key within the workflow configuration, as outlined in [this](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#using-an-environment) GitHub documentation.

# How could this be better?

It's clear that the root cause was my user error here. However, I definitely
think it is worth highlighting a couple of opportunities for improvement that
could prevent others from having to climb out of the same trap.

#### Better Error Messaging

Error messages could always be better right? When it comes to improving the feedback
loop, the first thing I identify is whether or not the application generating the error
message has enough information to actually generate a more descriptive message and/or
a suggested fix.

In this situation, that is totally the case. It was clear here that the secrets being
referenced did in fact exist, but just in an environment and not at the repository level.
GitHub even has [documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets#naming-your-secrets) on how two secrets with the same key but exist in both levels
are treated, so awareness of secrets in both places is already being handled to some
degree. 

To illustrate, imagine the debug logs had entries like this instead:

```
...

##[debug]..Evaluating secrets:
##[debug]..=> Object
##[debug]..Evaluating String:
##[debug]..=> 'DOCKERHUB_TOKEN'
##[debug]..Checking repository secrets
##[debug]=> null
##[debug]..Checking environment: staging
##[debug]=> null
##[debug]..Checking environment: production
##[debug]...Match found. Did you mean to reference an environment secret instead?

...
```

With this verbiage, there is no need to dive in for triaging. Users would
know immediately why their secrets were not accessible.

In a similar vein, it's worth asking the question whether referencing secrets
when there are 0 secrets detected is ever an expected scenario. In this circumstance,
the workflow couldn't find any secrets, despite both of them being listed. Perhaps
an error message indicating the following would be beneficial:

```
...

##[debug]..Evaluating secrets:
##[debug]..=> Object
##[debug]..No secrets exist

...
```

This message would at least be more straightforward that no secrets are detected,
eliminating the additional steps the user would need to take to understand why
`null` is being passed around.

#### Comprehensive Documentation

This one may be more of a reach, but it's worth calling out that the documentation
for the `docker/login-action` assumes that environments aren't being used. I don't
fault the maintainers for thinking it wouldn't be necessary to address explicitly -
after all, GitHub environments is it's own wide-spread feature, and it shouldn't
be necessary for every maintainer of a GitHub action to eductate users of other
GitHub features.

However, the maintainers _are_ responsible for writing the code to generate
error messages for their action. Their documentation doesn't outline the potential
error that can be thrown, or potential troubleshooting steps.
