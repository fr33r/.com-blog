+++
title = "Etags: The Secret Sauce to Scalable & Concurrent APIs"
date = "2023-04-08T10:35:13-07:00"
author = "fr33r"
cover = ""
tags = ["api", "web", "scaling"]
keywords = ["api", "web", "etag", "http", "rfc2616", "rfc7262", "header", "concurrency", "scale"]
description = "An in-depth walkthrough about what entity tags are and how you can leverage them to build scalable & highly concurrent RESTful APIs."
showFullContent = false
readingTime = true
hideComments = false
+++

If you are a software engineer in the web development space, I'm willing to
wager you have at least familiarized yourself with entity tags, or "etags" for
short. However, not many have fully seized the various opportunities etags
provide when implementing RESTful, HTTP-based APIs.

Incidentally I have become somewhat passionate about etags and
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
time, or both. - [RFC 7232 Section 2.3][rfc7232-section-2.3]

In other words, an entity tag is indicative of a particular state or "version"
of resource state communicated via a representation. Any time the
representation for the resource changes, the corresponding entity tag will
change. These entity tags are communicated via the [`Etag` header][etag] in HTTP
responses.

Now that we know what they are, it's time to get a little more concrete. After
all, what does one of these look like?

An entity tag consists of two main sections: an optional weakness identifier
(if the entity tag is a weak entity tag; more on that shortly), and a series
of [ASCII encoded][ascii] characters surrounded by double (`"`) quotes. The
second portion is often referred to as an "opaque string", "opaque validator",
or "entity tag value", simply meaning it's an abstract sequence of characters
typically without descernable or ulterior meaning.

### Strong

Stating that an entity tag is a "strong entity tag" really means that the
entity tag should be used to perform [strong validation][strong-validation]. Strong validation
is used to determine whether two representations of a resource are byte for byte
identical. This strict form of validation is required for a subset of
conditional HTTP headers, and in general provides the highest level of
confidence that the data being compared is equivalent.

#### Examples

```

"e6139f93eb47c9fe7eb7fc5ddb586511"
"3b6f421e7550395e28e091c5565ac80a"
"00a809937eddc44521da9521269e75c6"
```

### Weak

Contrasting with strong entity tags, a weak entity tag refers to one that
leverages [weak validation][weak-validation]. This form of validation does not assert byte for
byte equality, and instead relies on determining whether the two
representations are semantically equivalent. This relies on the engineer to
define these semantics. A weak entity tag is designated by
adding a `W/` prefix to the entity tag value.

#### Examples

```
W/"2b00042f7481c7b056c4b410d28f33cf"
W/"4ecc28c187a45859e7631bb871a41a7d"
W/"28198b369067e88dab9fefe85484dbf4"
```

## Which Type Should I Use?

Based on my experience and systems I've built over the years, I recommend
using **strong etags**. In the next section, I'll go into more detail on how
entity tags can be generated, but in general, strong entity tags are typically simpler
to generate from a coding perspective. Additionally, they leave the door
open for your API to support conditional request behaviors that rely on byte
for byte equality, such as [`If-Range`][if-range], which can be used to request a portion
of a resource representation. They are easier to reason about given
their generation algorithms are well known, documented, and widely available. Lastly,
they gracefully support content negotiation use cases given different
negotiated representations will natively have different content.

That said, it's worth highlighting the most prominent downside:
the slightest difference results in a different entity tag, even if the
data being conveyed in the representation is the same.

For example, if your representations leverage a data exchange format such as [JSON][json],
any changes to the ordering of the keys would result in a mismatch. Perhaps your
representation contains a collection or list; if the ordering of the elements
is different, again the entity tags will not match. Such fragility is as
burdensome as is the ability to tame these occurrences. For example, if you
can easily ensure JSON keys are serialized in a consistent order, that lists
are serialized in a consistent order, etc. than you should be able to safely
avoid unintended mismatches.

> **NOTE:** Some web frameworks already take liberties in how data is serialized
that help ensure the process is deterministic. For example, it's common to see
serialization functionality be implemented such that it sorts JSON keys in
alphabetical order. As such, the fragility described above is typically
already handled to some degree from the start.

## How Should I Generate Them?

Mechanically, entity tags are typically generated using well known, one-way
hashing algorithms such as [MD5][md5-algorithm] or [SHA256][sha2-algorithm].
The input to the hashing algorithm hinges on whether strong or weak validation
is desired.

When generating strong entity tags, the input is the serialized
representation. In other words, all of the bytes that comprise the
representation to be returned in a response to a [`GET`][get] request for the resource
are what ultimately gets hashed.

On the flip side, if weak validation is desired, you as the engineer have to
define what data is used as input. For example, perhaps only a subset of the
data items returned in a representation need to be considered - this could
be accomplished by serializing just those properties and using that result as input.

> **NOTE:** You may also see or hear the term 'variant' used interchageably with
'representation'. Some web frameworks use this terminology as well in their
code. Within the context of this article, you can assume they are synonymous
with one another.

It's important to reiterate that entity tags should be generated for each
representation for a resource, such that different representations for that
resource can be properly distinguished. This includes [content negotiation][content-negotiation]
as well.

If your HTTP API supports content negotiation, make sure that you are capable
of generating entity tags such that they factor in the variants your API supports.
For generating entity tags that support strong validation, this should be supported
out of the box. Representations that differ in language, encoding, or media type
will be comprised of different bytes. When generating entity tags that use
weak validation, these negotiable characteristics of each variant should
be used as input in combination with the semantics already put in place to
generate the entity tag.

## Storage

Although various request flows that leverage entity tags can operate completely
without storing entity tags and compute them on the fly prior to usage, I strongly
urge that you consider storing them. In absence of storing them, the
origin server basically needs to reconstitute the resource and serialize it
to determine what the current relevant entity tag is which
is costly. Additionally, data store technologies provide affordances that
can help ensure race conditions are handled gracefully.

### Considerations

There are some key considerations to weigh when choosing to store entity tags:

- In the event your API supports content negotiation, you will need to store the
dimensions of negotiation along with your entity tags such that the correct
one is selected. Examples of dimensions are media type, language, and encoding.
- You will need to use optimistic concurrency controls. Check to see if your database
of choice or web framework supports optimistic concurrency out of the box. If not,
your API will need to implement this.
- The metadata utilized for the optimistic concurrency control algorithm should
be stored directly with the entity tags. In other words,
judge whether the resource change in progress should pass or fail based on whether
a concurrent change to the stored entity tag has occurred.
- Entity tags should be generated within the same transaction as the resource
creation or modification operation. If your data store doesn't support explicit
transactions, I'd suggest creating a separate "view" of the resource state such
that it is all consolidated into one location. For example, if using a document
database, you could denormalize all resource state into
a single document. This way, all data used as input for the entity tag is
consolidated. This creates a single pathway of change that can be used to trigger
new entity tag creation, and optimistic concurrency control can be used to ensure
that a concurrent change has not occurred.

### Example (Relational DB)

Let's assume we are using a relational database solution such as [MySQL][mysql] or
[PostgreSQL][postgresql]. Since these solutions offer support for transactions we could
create a separate `entity_tags` table to store our entity tags. We then would
issue `INSERT` or `UPDATE` SQL statements to this table within the same
transaction as the resource state changes.

New table; ✅. We next need to decide what
data this table should store. The following are various data items we could
add to the schema of this table:

| Data Item | Description
| --- | --- |
| `uri` | The URI of the resource. |
| `media_type` | The [MIME][mime] media type of the representation. |
| `language`* | The language of the representation. |
| `encodings`* | The encodings of the representation. |
| `etag` | The entity tag. |
| `version` | The version column utilized to support optimistic concurrency. |

>  `*` = Data item may not be required; depends on the dimensions available for content negotiation.

Let's get to it! We could achieve the above with something similar to the following SQL:

{{< code language="sql" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
CREATE TABLE entity_tags
  (
     uri        VARCHAR NOT NULL,
     media_type VARCHAR NOT NULL,
     language   VARCHAR NOT NULL,
     encodings  VARCHAR NOT NULL,
     etag       VARCHAR NOT NULL,
     version    INTEGER NOT NULL,

     PRIMARY KEY (uri, media_type, language, encodings)
  ); 
{{< /code >}}

Within this table in place, some examples of records in this table could look
like this:

| `uri`         | `media_type`                   | `language` | `encodings` | `etag`                               | `version` |
| ------------- | ------------------------------ | ---------- | ----------- | ------------------------------------ | ----------|
| `/examples/1` | `application/vnd.example+json` |    `en-US` |      `gzip` | `"8afc1e9bec810034dafd45c6854f1dd9"` |         1 |
| `/examples/1` | `application/vnd.example+yaml` |    `en-US` |      `gzip` | `"87a1ca13ae46d84174ee2b607f03e36d"` |         3 |
| `/examples/1` | `application/vnd.example+json` |    `fr-CA` |      `gzip` | `"a4bef372fd7a743fb81ead93498ce414"` |         1 |
| `/examples/1` | `application/vnd.example+yaml` |    `fr-CA` |      `gzip` | `"937de310a05a530fa4c802d802020fdb"` |         4 |
| `/examples/1` | `application/vnd.example+yaml` |    `en-US` |      `gzip` | `"26e28f0c78c13daad592c8977afca45e"` |         1 |
| `/examples/2` | `application/vnd.example+json` |    `en-US` |      `gzip` | `"5d83f25b2cb0df85c52620eda1031e2c"` |         2 |
| `/examples/2` | `application/vnd.example+yaml` |    `en-US` |      `gzip` | `"57d4cc6b026949a45688c6352d88d023"` |         7 |
| `/examples/2` | `application/vnd.example+json` |    `fr-CA` |      `gzip` | `"337943905ca0bfe204b69eab51fff1a8"` |         3 |
| `/examples/2` | `application/vnd.example+yaml` |    `fr-CA` |      `gzip` | `"48dd5c5c560dabcf685899e0b2a5d866"` |         1 |
| `/examples/2` | `application/vnd.example+yaml` |    `en-US` |      `gzip` | `"6833d9d1ce9ec568f370572f6e029425"` |         2 |
|           ... |                            ... |        ... |         ... |                                  ... |       ... |


## Scaling Reads

The most popular application of entity tags is to help scale API read operations,
such as `GET` requests. This is achieved using what is referred to as
"conditional" `GET` requests.

### Conditional `GET`

Any request is classified as conditional if it provides any of these
conditional headers:

- [`If-Match`][if-match]
- [`If-None-Match`][if-none-match]
- [`If-Modified-Since`][if-modified-since]
- [`If-Unmodified-Since`][if-unmodified-since]
- [`If-Range`][if-range]

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
"refresh" the entry][conditional-update]
only if it has changed.

{{< image src="/img/conditional-get.gif" alt="Conditional GET Example" position="center" style="border-radius: 8px;" >}}

## Resource Integrity

The lesser known use case for conditional requests is to maintain resource
integrity. Using entity tags and conditional headers
such as `If-Match`, the origin server can utilize [optimistic concurrency][optimistic-concurrency]
control practices to validate whether the clients write request is being based
off of a stale representation, and properly facilitate a feedback cycle to
inform the client of this change of state.

Even though this has value in non-concurrent request patterns, it really
shines when it comes to modern systems where more than one client could be
modifying resource state at the same time. If implemented correctly, writes
to resource state will avoid the [Lost Update Problem][lost-update-problem], and clients will be
empowered to understand when relavent resource state changes outside the
scope of their interaction.

### Conditional `PUT` / `PATCH`

Modifications to resource state are achieved using the [`PUT`][put] and [`PATCH`][patch]
HTTP methods. When issuing conditional `PUT` or `PATCH` requests, the client
provides a value for the `If-Match` or `If-Unmodified-Since` headers. Again,
we will set `If-Unmodified-Since` to the side.

The premise here is similar to conditional `GET` requests, in the way that
clients indicate one or many entity tags they have recorded for the resource
they wish to interact with. The format of the `If-Match` header should
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

> **NOTE:** The `*` value essentially forces the request to succeed. More concretely, it
results in the precondition to evalute to `true` [as long as the resource
being targeted exists][if-match-directives]. Utilize this value with caution; it basically disables
the safety that this conditional request is intending to provide.

{{< image src="/img/conditional-update.gif" alt="Conditional GET Example" position="center" style="border-radius: 8px;" >}}

## Advanced

### Synchronizing with Code Changes

As stated previously, there are significant benefits to storing entity tags
instead of computing them on the fly prior to comparisons. One drawback of this
approach, however, is that stored entity tags become stale as the structure (and thus the
code responsible for generating that structure) of representations evolves.
Many changes happen organically over time that are backwards compatible to
API consumers, thus can be made inline to existing contracts.

There is room for creativity on how to handle this. One approach could be to
store a hash of the file contents responsible for generating the representation
alongside each entity tag stored. This would enable programmatic detection of
which entity tags need to be refreshed, which could be done lazily if detected
during a comparison operation. Instead of the lazy approach, if use cases
allow, a background job could be spawned upon code deployment or perhaps on a
cadence that identify these stale entity tags and update them accordingly.

Depending on the number of resources your system manages, a simpler solution
of regenerating all entity tags could be viable. The main benefit of this
approach is simplicity; there would be no need to attempt to track & handle file
content changes.

### Server-side Caching

The use case of leveraging entity tags to ensure client-side caches remain fresh
through conditional `GET` requests is well known. However, the opportunity for
server-side caching arises once the approach of storing entity tags is implemented.

When entity tags are stored, the resource doesn't need to be loaded prior
to determining the latest relevant entity tag. Loading the resource state
would defeat the point of the cache; the cache would be intended to return a
response without loading the resource with the goals of providing speedier
responses and reducing server system load.

Assuming we have built the supporting infrastructure to store entity tags,
including the appropriate mechanics that ensures entity tags are regenerated
atomically when resource state changes, a [cache busting][cache-busting]
strategy can be used. The cache key would itself comprise of the entity tag,
and since changes to resource state would result in a new entity tag
altogther, new representations are stored as completely different cache
entries. This strategy eliminates the possibility where a stale cached
representation is used incidentally.

{{< image src="/img/server-side-caching.png" alt="Server-side Caching Example" position="center" style="border-radius: 8px;" >}}

Given new entries will be made in the cache each time the entity tag
changes, I suggest using a reasonably short expiration. Selecting a timeframe
for your expiration will be dependent on your use cases, the balance being
between growing cache size vs. cache hit count.

> **NOTE:** To determine a cache entry expiration, think about how much time
typically elapses between creation of the resource and the first time it is
retrieved. Additionally, if your API allows modification of your resources,
it's similarly valuable to consider how much time typically elapses between
the modification and a subsequent retrieval. Lastly, if you resources are
typically shared amongst many clients, consider how much time elapses between
the retrievals each client issues.

### Scaling Collection-based Searches

We went in depth on how entity tags could be used to maintain client-side caches
of resources, as well as maintain resource integrity during modifications -
but what about collection resources? If resource state changes for a resource that
also influences the state of a collection resource, how do we make sure the
entity tag for that collection gets updated to?

Some collection resources can be conceptually huge. Often APIs offer filtering
or pagination semantics that make consuming these collections easier and more
useful for clients. Given this, we should rule out the approach of attempting
to generate a new entity tag based on the entire collection. Instead, the
entity tag should be based on the representation actually sent to the client;
pagination and filtering included.

From here, we should identify common searches or paginations requests that the
clients of the API issue. Perhaps these clients are commonly searching the
collection resource based on an identifier or other high cardinality
characteristic, such as a timestamp. Maybe it's common for the client to only
be interested in the first one or two pages of the results. These workflows
will identify which we should actively attempt to cache.

Once we've identified these common requests, we can actually pregenerate these
representations and their respective entity tags when any dependent resource
state is modified. In other words, the existing logic in place to update
the entity tags for a single resource can be further expanded to also not only
generate the entity tag based on the representation for that single resource,
but also a finite list of representations for dependent collection resources.

With these new entity tags stored, it's not possible for us to store these
precomputed search or pagination results in our cache.

## Conclusion

Who knew there was so much to know about entity tags, right? It's pretty amazing
that so many capabilities can be unlocked if they are fully adopted. My hope is that
you walk away from this feeling empowered to leverage entity tags on APIs you
are building!

Do you currently, or have you previously used entity tags in projects you've
been a part of? Have you ever used them in a way that I didn't capture?
Was there anything in this article I could go deeper on, or that
you think I could improve? Let me know in the comments!

[rfc7232-section-2.3]: https://www.rfc-editor.org/rfc/rfc7232#section-2.3
[rfc2616-section-1.3]: https://www.rfc-editor.org/rfc/rfc2616#section-1.3
[md5-algorithm]: https://en.wikipedia.org/wiki/MD5
[sha2-algorithm]: https://en.wikipedia.org/wiki/SHA-2
[if-range]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Range
[if-match]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match
[if-none-match]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match
[if-modified-since]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since
[if-unmodified-since]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Unmodified-Since
[if-match-directives]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match#directives
[ascii]: https://en.wikipedia.org/wiki/ASCII
[strong-validation]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#strong_validation
[weak-validation]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#weak_validation
[optimistic-concurrency]: https://en.wikipedia.org/wiki/Optimistic_concurrency_control
[etag]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag
[content-negotiation]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
[cache-busting]: https://www.keycdn.com/support/what-is-cache-busting
[lost-update-problem]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#avoiding_the_lost_update_problem_with_optimistic_locking
[json]: https://en.wikipedia.org/wiki/JSON
[mime]: https://en.wikipedia.org/wiki/MIME
[postgresql]: https://www.postgresql.org/
[mysql]: https://www.mysql.com/
[conditional-update]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests#cache_update
[put]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT
[patch]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH
[get]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET
