+++
title = "Etags"
date = "2023-04-08T10:35:13-07:00"
author = "fr33r"
cover = ""
tags = ["api", "web", "scaling"]
keywords = ["api", "web", "etag", "http", "rfc2616", "rfc7262", "header", "concurrency", "scale"]
description = ""
showFullContent = false
readingTime = true
hideComments = false
+++

If you are a software engineer in the web development space, I'm willing to
wager you have at least familiarized yourself with entity tags, or "etags" for
short. However, not many have fully seized the various opportunities etags
provide when implementing RESTful, HTTP-based APIs.

Interestingly, I've incidentally become somewhat passionate about etags and
their applied usage as I've continued through my career. They are often a
critical necessity that gets overlooked when building scalable, concurrent
systems for the web. Having knowledge of what they are and how to use them
will serve you well by ultimately empowering you and others alongside you to
build robust APIs that maintain resource integrity and can scale read flows
to great volumes.

## WTF are etags?

> An entity tag is an opaque validator for differentiating between multiple
representations of the same resource, regardless of whether those multiple
representations are due to resource state changes over time, content
negotiation resulting in multiple representations being valid at the same
time, or both. - [RFC-7232 Section 2.3][rfc7232-section-2.3]

In other words, an entity tag is indicative of a particular state or "version"
of resource state communicated via a representation. Any time the
representation for the resource changes, the corresponding entity tag will
change. These entity tags are communicated via the `Etag` header in HTTP
responses.

Now that we know what they are, it's time to get a little more concrete. After
all, what does one of these look like?

An entity tag consists of two main sections: an optional weakness identifier
(if the entity tag is a weak entity tag; more on that shortly), and a series
of [ASCII encoded][ascii] characters surrounded by double (`"`) quotes. The
second portion is often referred to as an "opaque string", "opaque validator",
or "entity tag value", simply meaning it's an abstract sequence of charactors
typically without descernable alterior meaning. The first section is used to
determine whether the entity tag is classified as either strong, or weak.

### Strong

Stating that an entity tag is a "strong enitity tag" really means that the
entity tag should be used to perform strong validation. Strong validation
is used to determine whether two representations of a resource at byte to byte
identical. This strict form of validation is required for a subset of
conditional HTTP headers, and in general provides the highest level of
confidence that the data being compared is equivalent.

### Weak

Contrasting with strong entity tags, a weak entity tag refers to one that
leverages weak validation. This form of validation does not assert byte for
byte equality, and instead relies on determining whether the two
representations are semantically equivalent. This relies on the author of the
origin server to define these semantics.

### Examples

```
# entity tag using weak validation.
W/"2b00042f7481c7b056c4b410d28f33cf"

# entity tag using strong validation.
"e6139f93eb47c9fe7eb7fc5ddb586511"
```

## Which type should I use?

Based on my experience and systems I've built over the years, I recommend
using strong etags. In the next section, I'll go into more detail on how
entity tags can be generated, but in general, strong entity tags are simpler
to generate from a coding perspective. Additionally, they leave the door
open for your API to support conditional request behaviors that rely on byte
for byte equality, such as [`If-Range`][if-range], which can be used to request a portion
of a resource representation. Lastly, they are easier to reason about given
their generation algorithms are well defined and easily reproducible.

That said, it's worth highlighting some downsides; the most prominent being
that the slightest difference results in a different entity tag, even if the
data communicated in the representation is the same.

For example, if your representations are a data exchange format such as JSON,
any changes to the ordering of the keys would result in a mismatch. Perhaps your
representation contains a collection or list; if the ordering of the elements
is different, again the entity tags will not match. Such fragility is as
burdensome as is the ability to tame these occurances. For example, if you
can easily ensure JSON keys are serialized in a consistent order, that lists
are serialized in a consistent order, etc. than you should be able to safely
avoid unintended mismatches.

## How should I generate them?

Mechanically, entity tags are typically generated using well known, one-way
hashing algorithms such as [MD5][md5-algorithm] or [SHA256][sha2-algorithm].
The input to the hashing algorithm hinges on whether strong or weak validation
is desired.

When generating strong entity tags, the input is the serialized
representation. In other words, all of the bytes that are intended for the
representation to be returned in a response to a `GET` request for the resource
are what gets hashed.

On the flip side, if weak validation is desired, you as the engineer have to
define what data is used as input. For example, perhaps only a subset of the
data items returned in a representation need to be considered - this could
be accomplished by serializing just those properties and using that result as input.

### What are variants?

> A resource may have one, or more than one, representation(s) associated with
it at any given instant. Each of these representations is termed a 'variant'.
Use of the term `variant' does not necessarily imply that the resource is
subject to content negotiation. - [RFC 2616 Section 1.3][rfc2616-section-1.3]

## Storage

Although various API flows that leverage entity tags can operate completely
without storing entity tags and compute them on the fly prior to usage, I strongly
urge that you consider storing them. In absence of storing them, the
origin server basically needs to reconsitute the resource and serialize it
to determine what the current relevant entity tag is for a resource which
is costly.

### What not to do

Since entity tags are fundamentally a means of tracking and communicating
"versions" of resource state, it can be tempting for engineers to reach
for a versioning solution included with various well-known web frameworks
and apply it directly to a database table responsible for storing resource
state.

Many of these implementations are based on storing a version as an additional
column in a database table. Don't do this. Why?

Many times, a resource representation includes data that is stored across
many tables or data sources. Storing a version at a table level means that
you would need to devise a complex solution for ensure that version gets
changed even if data ultimately stored in a different table is altered.

Also, not all data stored is included in resource representations. Storing
versions at a table level as many of these web frameworks enable results in
a version change even if the resulting resource representation wouldn't be
altered. A thrashing entity tag will reduce efficacy of various entity tag
applications, such as conditional `GET` and conditional `PUT` / `PATCH`
requests.

### RDBMS (SQL)

If you are building an API that leverages a relational database management
system (RDBMS), entity tags could be represented as a separate table
that is modified within the scope of the transaction responsible for executing
the SQL statements that create or modify the resource state itself. This
approach ensures that entity tags are consistent with the resource state being 
persisted.

The following is an example schema for such a table:

| Column Name | Data Type | Description
| --- | --- | --- |
| `uri` | `VARCHAR` | The URI of the resource. |
| `media_type` | `VARCHAR` | The MIME media type of the representation. |
| `etag` | `VARCHAR` | The entity tag. |
| `version` | `INTEGER` | The version column utilize to support optimistic locking. |

### Document storage (NoSQL)

## Scaling reads

The post popular application of entity tags is to help scale API read operations,
such as `GET` requests. This is achieved using what is referred to as
"conditional `GET` requests.

### Conditional `GET`

Any request is classified as conditional if it provides any of these
conditional headers:

- [`If-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match)
- [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)
- [`If-Modified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)
- [`If-Unmodified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Unmodified-Since)
- [`If-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Range)

More specifically, conditional `GET` requests most often utilize either the
`If-None-Match` or `If-Modified-Since` headers. Since we are mainly focusing
on entity tags in this dicussion, we'll focus just on `If-None-Match`.

In the context of conditional `GET` requests, the `If-None-Match` header
is used as a way for a client to conditionally request a fresh representation
for a resource only if that resource has an entity tag _that does not match_
any of the entity tags provided within the value of the header. The format
of that header looks like the following:

```
If-None-Match: "<entity tag value>"
If-None-Match: "<entity tag value>", "<entity tag value>", …
If-None-Match: *
```

HTTP clients (as well as intermediary hosts such as proxies) often maintain
caches that store responses to previous requests. Those clients maintain that
cache, and expiration of the entries are primarily controlled by the value
of the `Cache-Control` header sent in the response from the origin server. Once
the entry has expired, the [client issues these conditional `GET` requests to
"refresh" the entry](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#cache_update)
only if it has changed.

{{< image src="/img/conditional-get.gif" alt="Conditional GET Example" position="center" style="border-radius: 8px;" >}}

## Resource integrity

The lesser known use case for conditional requests is to maintain resource
integrity. Using entity tags and conditional headers
such as `If-Match`, the origin server can utilize optimistic concurrency
control practices to validate whether the clients write request is being based
off of a stale representation, and properly facilitate a feedback cycle to
inform the client of this change of state.

Even though this has value even in non-concurrent request patterns, it really
shines when it comes to modern systems where more than one client could be
modifying resource state at the same time. If implemented correctly, only
a single clients write will succeed and the remaining will be rejected with
[`412 Precondition Failed`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/412).

### Conditional `PUT` / `PATCH`

Modifications to resource state are achieved using the
[`PUT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)
and [`PATCH`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH)
HTTP methods. When issuing conditional `PUT` or `PATCH` requests, the client
provides a value for the `If-Match` or `If-Unmodified-Since` headers. Again,
we will set `If-Unmodified-Since` to the side.

The premise here is similar to conditional `GET` requests, in the way that
clients indicate one or many entity tags they have recorded for the resource
they wish to interacted with. The format of the `If-Match` header should
look familiar:

```
If-Match: "<entity tag value>"
If-Match: "<entity tag value>", "<entity tag value>", …
If-Match: *
```

However, as the header name communicates, the intention is the client
wishes to perform the modifications _only if the current resource entity tag
matches_ at least one of the provided values for `If-Match`. It is only then
that the modifications are performed, as this process validates the client
is issuing the request based on the context of the latest (most current)
state of that resource.

> NOTE: The `*` value essentially forces the request to succeed. More concretely, it
results in the precondition to evalute to `true` [as long as the resource
being targeted exists][if-match-directives]. Utilize this value with caution; it basically disables
the safety that this conditional request is intending to provide.

{{< image src="/img/conditional-update.gif" alt="Conditional GET Example" position="center" style="border-radius: 8px;" >}}

## Advanced

### Scaling collection-based searches

### Synchronizing with code changes

As stated previously, there are signficant benefits to storing entity tags
instead of computing them on the fly prior to comparisons. One drawback of this
approach is that stored entity tags become stale as the structure (and thus the
code responsible for generating that structure) of represntations evolves.
Many changes happen organically over time that are backwards compatible to
API consumers.

There is room for creativity on how to handle this. One approach could be to
store a hash of the file contents responsible for generating the representation
alongside each entity tag stored. This would enable programmatic detection of
which entity tags need to be refreshed, which could be done lazily if detected
during a comparison operation. Instead of the lazy approach, if use cases
allow, a background job could be spawned upon code deployment or perhaps on a
cadence that identify these stale entity tags and update them accordingly.

Depending on the number of resources your system manages, a simpler solution
of regenerating all entity tags could be viable. The main benefit of this
approach is simplicity.

[rfc7232-section-2.3]: https://www.rfc-editor.org/rfc/rfc7232#section-2.3
[rfc2616-section-1.3]: https://www.rfc-editor.org/rfc/rfc2616#section-1.3
[md5-algorithm]: https://en.wikipedia.org/wiki/MD5
[sha-2-algorithm]: https://en.wikipedia.org/wiki/SHA-2
[if-range]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Range
[if-match-directives]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match#directives
