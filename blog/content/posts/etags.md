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
time, or both. - [RFC-7232 Section 2.3][rfc-7232-section-2.3]

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
be accomplished by serializing just those properties and using those as input.

> NOTE: 

Interestingly, documentation about entity tags typically discuss how
generating strong entity tags can prove burdensome, and may even go as far as
possibly impacting peformance. As with most things software engineering, there
are some variables and nuances to consider when interpreting this claim, and
ultimately some trade offs to evaluate based on your use cases. In fact, I'd
argue it's more combersome to explicitly define

>>>



### What are variants?

> A resource may have one, or more than one, representation(s) associated with
it at any given instant. Each of these representations is termed a `variant'.
Use of the term `variant' does not necessarily imply that the resource is
subject to content negotiation. - [RFC 2616 Section 1.3][rfc2616-section-1.3]

## Etag storage

### What not to do

### RDBMS (SQL)

### Document storage (NoSQL)

## Scaling reads

### Conditional `GET`

## Resource integrity

### Conditional `PUT`

### Conditional `PATCH`

## Advanced

### Scaling collection-based searches

### Synchronizing with code-changes

[rfc7232-section-2.3]: https://www.rfc-editor.org/rfc/rfc7232#section-2.3
[rfc2616-section-1.3]: https://www.rfc-editor.org/rfc/rfc2616#section-1.3
[md5-algorithm]: https://en.wikipedia.org/wiki/MD5
[sha-2-algorithm]: https://en.wikipedia.org/wiki/SHA-2
[if-range]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Range
