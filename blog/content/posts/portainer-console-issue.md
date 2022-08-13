+++
title = "You Had One Job Portainer Console!"
date = "2022-08-12T22:04:10-07:00"
author = "fr33r"
cover = "img/sonic-waiting.jpeg"
tags = ["portainer", "fix", "quickhit", "nginx", "devops"]
keywords = ["portainer", "nginx", "websockets"]
description = "A quick dive into why my Portainer console connectivity stopped working."
showFullContent = false
readingTime = true
hideComments = false
+++

I use [Portainer][portainer] as my central hub for managing my Docker Swarm
that powers my home lab, and it's awesome.

One of the many features it includes is the ability to open up a 'console' to any
container (task) running in the swarm - think basically a UI for
`docker exec -it <CONTAINER ID> /bin/bash`. It even provides the affordance of
specifying a different user (other than the default of `root`) or shell (other
than the default of `bash`).

# Connecting...

One day, I went to hop into a console session on one of my containers, but instead
of the shell launching, Portainer seemed to be stuck when attempting to connect.

{{< figure src="/img/portainer-connecting-issue.png" alt="Portainer Connecting Issue" position="center" caption="Screenshot of the issue." >}}

What gives? No error toasts appear in the top right to examine as one might expect.
Time to dig deeper.

Once I landed in the logs, I tripped over some of these:

```
...

level=info msg="2022/08/13 05:06:35 websocketproxy: Error when copying from backend to client: websocket: close 1006 (abnormal closure): unexpected EOF"
level=info msg="2022/08/13 05:07:01 websocketproxy: Error when copying from backend to client: websocket: invalid close code"

...

level=info msg="2022/08/13 05:12:22 websocketproxy: couldn't upgrade websocket: the client is not using the websocket protocol: 'upgrade' token not found in 'Connection' header"
level=info msg="2022/08/13 05:12:40 websocketproxy: couldn't upgrade websocket: the client is not using the websocket protocol: 'upgrade' token not found in 'Connection' header"

...

```

Interesting. Seems Portainer is attempting to make use of some websockets but can't.

# Root Cause + Fix

To take a half a step back real quick, it's worth mentioning that I use nginx as
my reverse proxy of choice for my swarm. Essentially it runs as a container
within the swarm and forwards all traffic on ports `80` (http) or `443` (https)
to other services. One of these services is Portainer.

As it turns out, my nginx configuration did not support websockets. I had to make
the following changes:


{{< code language="diff" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
server {
	server_name ~^sweet.domain.com$;

	listen 443 ssl;
	ssl_certificate /ssl/cert.pem;
	ssl_certificate_key /ssl/key.pem;

	access_log /var/log/nginx/data-access.log combined;

	location / {
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_pass https://sweet_swarm_task:9443;
+		proxy_http_version 1.1;
+		proxy_set_header Upgrade $http_upgrade;
+		proxy_set_header Connection "upgrade";
	}
}
{{< /code >}}

I definitely wasn't the only one to make this mistake; this
[GitHub issue][github-issue] being one example! Although, reading through
some of the conversation retriggered some interest in exploring
[Nginx Proxy Manager][nginx-proxy-manager] once again - perhaps more on that
in a future post.

#### Breakdown

In case you are saying:

> Hold up, what are these changes?

I got you covered. Let's break it down.

##### `proxy_http_version 1.1;`

Indicates that the HTTP version should be set to `1.1`. Connection upgrades
can only be performed for `1.1`, and not `1.0` or `2`
([explicitly disallowed][upgrade-mechanism]).

##### `proxy_set_header Upgrade $http_upgrade;`

Sets the value of the `Upgrade` [header][upgrade-header] to have the value
provided (see [`$http_*` nginx variables][nginx-http-vars]). Clients utilize
this header to indicate that they wish to upgrade the connection from a
particular version of HTTP (1.1 for example) to another version, or to a websocket
connection.

##### `proxy_set_header Connection "upgrade";`

Sets the `Connection` [header][connection-header] to have a value of
`"upgrade"`, which indicates that nginx needs to process the `Upgrade`
[header][upgrade-header] before forwarding. This header must be sent whenever
the `Upgrade` is sent.

# Further Reading

I've only scratched the surface on this stuff. If you are interesting in reading
further, check out [this awesome post][nginx-blog-post] on the
[nginx blog][nginx-blog].

[portainer]: https://www.portainer.io/
[github-issue]: https://github.com/portainer/portainer/issues/6353
[nginx-proxy-manager]: https://nginxproxymanager.com/
[connection-header]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection
[upgrade-header]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade
[nginx-blog-post]: https://www.nginx.com/blog/websocket-nginx/
[nginx-blog]: https://www.nginx.com/blog
[nginx-http-vars]:[https://nginx.org/en/docs/http/ngx_http_core_module.html#var_http_]
[upgrade-mechanism]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism#upgrading_to_a_websocket_connection
