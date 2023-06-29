%%%
Title = "RDAP Extensions"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "rdap-extensions"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-newton-regext-rdap-extensions-00"
stream = "IETF"
status = "informational"
date = 2023-06-23T00:00:00Z

[[author]]
initials="A."
surname="Newton"
fullname="Andy Newton"
organization="ICANN"
[author.address]
email = "andy@hxr.us"

[[author]]
initials="J."
surname="Singh"
fullname="Jasdip Singh"
organization="ARIN"
[author.address]
email = "jasdips@arin.net"

%%%

.# Abstract

This document describes the usage of extensions in RDAP.

{mainmatter}

# Background

The Registration Data Access Protocol (RDAP) defines a uniform means to access data
from Internet operations registries, specifically Domain Name Registries (DNRs) and
Internet Number Registries (INRs). The queries for DNRs and INRs are defined in
[@!RFC9082] and the responses for DNRs and INRs are defined in [@!RFC9083].

RDAP contains a means to define extensions for queries not found in [@!RFC9082] and
responses not found in [@!RFC9083].

# The RDAP Extension Identifier

[@!RFC7480, section 6] describes the identifier used to signify RDAP extensions
and the IANA registry into which RDAP extensions are to be registered.

In brief, RDAP extensions identifiers start with an alphabetic character and may
contain alphanumeric characters and "_" (underscore) characters. This formulation
was explicitly chosen to allow compatibility with variable names in programming
languages and transliteration with XML.

RDAP extension identifiers have no explicit structure and are opaque in that no
inner-meaning can be "seen" in them.

When in use in RDAP, extension identifiers are prepended to both URL path segments
and JSON attribute names. In both cases, the extension identifier acts as a namespace
preventing collisions between extension elements.

# Usage in Queries

[@!RFC9082, section 5] describes the use of extension identifiers in formulating
URIs to query RDAP servers. The extension identifiers are to be prepended to the
path segments they use. For example, if an extension uses the identifier
`foobar`, then the path segments used in that extension are prepended with `foobar_`.
If the `foobar` extension defines paths `fizz` and `fazz`, the URIs for this
extension might take the following form:

    https://base.example/foobar_fizz
    https://base.example/foobar_fazz

Although [@!RFC9082] describes the use of URI query strings, it does not define
their use with extensions. [@!RFC7480] instructs servers to ignore unknown query
parameters. Therefore, the use of query parameters, prefixed or not with an
extension identifier, is undefined. Despite this, there are several extensions 
that do specify query parameters.

# Usage in JSON

[@!RFC9083, section 2] describes the use of extension identifiers in the JSON
returned by RDAP servers. Just as in URIs, the extension identifier is prepended
to JSON names to create a namespace so that the JSON name from one extension
will not collide with the JSON name of another extension. And just as with
URIs, clients are to ignore unknown JSON names.

The example given in [@!RFC9083] is as follows:

    {
      "handle" : "ABC123",
      "lunarNIC_beforeOneSmallStep" : "TRUE THAT!",
      "remarks" :
      [
        {
          "description" :
          [
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "lunarNIC_harshMistressNotes" :
      [
        "In space,",
        "nobody can hear you scream."
      ]
    }

In this example, the extension identified by `lunarNIC` is prepended
to the names of both a JSON string and a JSON array.

The following example shows this use with a JSON object.

    {
      "handle" : "ABC123",
      "remarks" :
      [
        {
          "description" :
          [
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "lunarNIC_author" :
      {
        "firstInitial": "J",
        "lastName": "Heinlein"
      }
    }

Here the JSON name "lunarNic_author" will separate the JSON from other
extensions that may have an "author" structure. But the JSON contained
within "lunarNIC_author" need not be prepended as the extension collision
is avoided by "lunarNIC_author".

# Camel Casing

The styling convention used in [@!RFC9083] for JSON names is often called
"camel casing", in reference to the hump of a camel. In this style, the 
first letter of every word, except the first word, composing a name is capitalized.
This convention was adopted to visually separate the namespace from the
name, with an underscore between them.

Though there is no explicit guidance to use camel case names, extensions would
be wise to continue the style.

# Two Classes of Extensions

Though all RDAP extensions are to be registered in the IANA RDAP extensions
registry, there is an implicit two-class system of extensions that comes from
the inherit ownership of the RDAP specifications by the IETF: extensions
created by the IETF and extensions not created by the IETF.

In the perspective of how extensions identifiers are used as namespace
separators, extensions created by the IETF are not required to be prefixed
with an extension identifier as the IETF can coordinate its own activities
to avoid name collisions. In practice, extensions owned by the IETF do use
extension identifiers.

# Extension Versioning

Because RDAP extensions are opaque, they posses no explicit version despite
the fact that some extension identifiers include trailing numbers. That
is, RDAP extensions are opaquely versioned.

For example, `fizzbuzz_1` may be the successor to `fizzbuzz_0`, but it
may also be an extension for a completely separate purpose. Only consultation
of the definition of `fizzbuzz_1` will determine its relationship with
`fizzbuzz_0`. Additionally, `fizzbuzz_99` may be the predecessor of `fizzbuzz_0`.

## Backwards-Compatible Changes

If an RDAP extension author wants to publish a new version of an
extension that is backwards-compatible with the previous version, then
one option is for the new version of the extension to define a new
identifier, as well as requiring that both the previous conformance
code and the new conformance code be included in responses.  That way,
clients relying on the previous version of the extension will continue
to function as intended, while clients wanting to make use of the
newer version of the extension can check for the new conformance code
in the response.

This approach can be used for an arbitrary number of new
backwards-compatible versions of a given extension.  For an extension
with a large number of backwards-compatible successor versions, this
may lead to a large number of conformance codes being included in
responses.  An extension author may consider excluding older
conformance codes from the set required by new successor versions,
based on data about client use/support or similar.

## Backwards-Incompatible Changes

With the current extension model, an extension with a
backwards-incompatible change is indistinguishable from a new,
unrelated extension.  Implementors of such changes should consider the
following:

 - whether the new version of the extension can be provided alongside
   the old version of the extension, so that a service can simply
   support both during a transition period;
 - whether some sort of client signalling should be supported, so that
   clients can opt for the old or new version of the extension in
   responses that they receive (see
   [@!I-D.newton-regext-rdap-x-media-type] for an example of how this
   might work); and
 - whether the extension itself should define how versioning is
   handled within the extension documentation.

## Other

[@!I-D.gould-regext-rdap-versioning] is an extension that provides a
more full-featured approach to versioning of RDAP extensions than that
supported by the base RDAP specifications.  Depending on an extension
author's use case, the approach set out in this document may be
preferable to one that relies on the base specifications alone.

# Extension Definitions

Extensions must be documented in an RFC or in some other permanent and readily
available reference, in sufficient detail that interoperability between independent 
implementations is possible.

Though RDAP gives each extension its own namespace, the definition of an
extension may re-use definitions found in the base RDAP specification or in any
other properly registered extension.

[@!RFC9083] notes that the extension identifiers provide a "hint" to the client
as to how to interpret the response. This wording does not intentionally restrict
the extension to defining only JSON values within the extensions namespace.
Therefore, an extension may define the use of its own JSON values together
with the use of JSON values from other extensions or RDAP specifications. As with
the ICANN profile or RIR profile extensions, the extension may simply signal 
policy applied to already defined RDAP structures.

# Existing Extension Registrations

The following extensions have been registered with IANA, but do not
comply with the requirements set out in the base specifications, as
clarified by this document:

 - Extension identifier: fred
    - RDAP conformance value: fred\_version\_0
    - Field/path prefix: fred

 - Extension identifier: artRecord
    - RDAP conformance value: artRecord\_level\_0
    - Field/path prefix: artRecord

 - Extension identifier: platformNS
    - RDAP conformance value: platformNS\_level\_0
    - Field/path prefix: platformNS

 - Extension identifier: regType
    - RDAP conformance value: regType\_level\_0
    - Field/path prefix: regType

Client authors should be aware that responses that make use of these
extensions may require special handling on the part of the client.
Also, while these extensions will be retained in the registry, future
extensions that are similarly noncompliant will not be registered.

To avoid any confusion with the operation of the existing entries, an
extension registration that attempts to use one of the RDAP
conformance values given in this section as an extension identifier
(and so as an RDAP conformance value also) will be rejected.

{backmatter}
