%%%
Title = "RDAP Extensions"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "rdap-extensions"
ipr= "trust200902"
updates = [7480, 9082, 9083]

[seriesInfo]
name = "Internet-Draft"
value = "draft-ietf-regext-rdap-extensions-02"
stream = "IETF"
status = "standard"
date = 2024-07-26T00:00:00Z

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

[[author]]
initials="T."
surname="Harrison"
fullname="Tom Harrison"
organization="APNIC"
[author.address]
email = "tomh@apnic.net"

%%%

.# Abstract

This document describes and clarifies the usage of extensions in RDAP.

{mainmatter}

# Background

The Registration Data Access Protocol (RDAP) defines a uniform means to access data
from Internet operations registries, specifically Domain Name Registries (DNRs) and
Internet Number Resource Registries (INRRs). The queries for DNRs and INRRs are defined in
[@!RFC9082] and the responses for DNRs and INRRs are defined in [@!RFC9083].

RDAP contains a means to define extensions for queries not found in [@!RFC9082] and
responses not found in [@!RFC9083]. RDAP extensions are also described in [@!RFC7480].
This document uniformly describes RDAP extensions, clarifies their usage, and
defines additional semantics that were previously undefined.

# The RDAP Extension Identifier {#extension_identifier}

[@!RFC7480, section 6] describes the identifier used to signify RDAP extensions
and the IANA registry into which RDAP extensions are to be registered.

In brief, RDAP extensions identifiers start with an alphabetic character and may
contain alphanumeric characters and "_" (underscore) characters. This formulation
was explicitly chosen to allow compatibility with variable names in programming
languages and transliteration with XML.

When in use in RDAP, extension identifiers are prepended to URL path segments,
URL query parameters, and JSON object member names (herein further referred to 
as "JSON names").  In all cases, the extension identifier acts as a namespace
preventing collisions between extension elements.

Additionally, implementers and operators can use the extension identifiers
to find an extensions definition (via the IANA registry).

RDAP extension identifiers have no explicit structure and are opaque in that no
inner-meaning can be "seen" in them. This document restricts the syntax
of RDAP extension identifiers from containing two consecutive "_" (underscore)
characters and reserves their use for the future definition of structure (such
as to define a versioning scheme). That is, RDAP extensions MUST NOT define
an identifier with two consecutive underscore characters ("__") unless explicitly
adhering to an RFC describing such usage.

RDAP extensions MUST NOT define an extension identifier that when prepended to
an underscore character may collide with an existing extension identifier.
For example, if there were a pre-existing identifier of "foo_bar", another extension 
could not define the identifier "foo". Likewise, if there were a pre-existing
identifier of "foo_bar", another extension could not define the identifier "foo_bar_buzz".
However, an extension could define "foo" if "foobar" pre-existed and vice versa.

For this reason, usage of an underscore character in RDAP extension identifiers
is NOT RECOMMENDED. Implementers should be aware that many existing extension
identifiers do contain underscore characters.

[@!RFC7480] does not explicitly state that extension identifiers are case-sensitive.
This document updates the formulation in [@!RFC7480] to explicitly note that extension
identifiers are case-sensitive, and extension identifiers MUST NOT be registered
where a new identifier is a mixed-case version of an existing identifier. For example,
if "lunarNIC" is already registered as an identifier, a new registration with "lunarNic"
(note the lowercase if "ic" in "Nic") is not allowed.

# Usage in Queries {#usage_in_queries}

[@!RFC9082, section 5] describes the use of extension identifiers in formulating
URIs to query RDAP servers. The extension identifiers are to be prepended to the
path segments they use. For example, if an extension uses the identifier
"foobar", then the path segments used in that extension are prepended with "foobar_".
If the "foobar" extension defines paths "fizz" and "fazz", the URIs for this
extension might take the following form:

    https://base.example/foobar_fizz
    https://base.example/foobar_fazz

While [@!RFC9082] describes the extension identifier as a prepended string to a
path segment, it does not describe the usage of the extension identifier as path
segment which may have child path segments. This document updates [@!RFC9082] to
allow the usage of extension identifiers as path segments which may have child path
segments. For example, if the "foobar" extension defines the child paths "fizz" and "fazz",
the URIs for this extension would take the following forms:

    https://base.example/foobar/fizz
    https://base.example/foobar/fazz

Extensions defining new URI paths MUST explicitly define the expected response
to each new URI path. New URI paths may return existing object classes or search
results as defined in [@!RFC9083], object classes or search results defined by
the extension (see (#object_classes_in_extensions) and (#search_results_in_extensions)
below), or object classes or search results from other extensions.

Although [@!RFC9082] describes the use of URI query strings, it does not define
their use with extensions. [@!RFC7480] instructs servers to ignore unknown query
parameters. Therefore, the use of query parameters, prefixed or not with an
extension identifier, is undefined as defined in [@!RFC9082] and [@!RFC7480]. 

Despite this, there are several extensions that do specify query parameters.
This document updates [@!RFC9082] with regard to the use of RDAP extension
identifiers in URL query parameters. 

When an RDAP extension defines query parameters to be used with a URL path
that is not defined by that RDAP extension, those query parameter names SHOULD be 
constructed in the same manner as URL path segments (that is, extensions ID + '_' 
+ parameter name). See section (#extension_classes) regarding when usage of an 
extension identifier is required.

When an RDAP extension defines query parameters to be used with a URL path defined
by that RDAP extension, prefixing of query parameters is not required.

See (#redirects) and (#referrals) for other guidance on the use of query
parameters. 

# Usage in JSON {#usage_in_json}

[@!RFC9083, section 2] describes the use of extension identifiers in the JSON
returned by RDAP servers. Just as in URIs, the extension identifier is prepended
to JSON names to create a namespace so that the JSON name from one extension
will not collide with the JSON name of another extension. And just as with
unknown query parameters in URIs, clients are to ignore unknown JSON names.

The example given in [@!RFC9083] is as follows:

    {
      "handle": "ABC123",
      "lunarNIC_beforeOneSmallStep": "TRUE THAT!",
      "remarks":
      [
        {
          "description":
          [
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "lunarNIC_harshMistressNotes":
      [
        "In space,",
        "nobody can hear you scream."
      ]
    }

In this example, the extension identified by "lunarNIC" is prepended
to the names of both a JSON string and a JSON array.

As [@!RFC9083, section 4.1] requires the use of the `rdapConformance` data structure
and the `objectClassName` string is required of all object class instances,
the complete example from above would be:

    {
      "rdapConformance": [
        "rdap_level_0",
        "lunarNIC"  
      ],
      "objectClassName": "domain",
      "handle": "ABC123",
      "ldhName": "example.com",
      "lunarNIC_beforeOneSmallStep": "TRUE THAT!",
      "remarks":
      [
        {
          "description":
          [
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "lunarNIC_harshMistressNotes":
      [
        "In space,",
        "nobody can hear you scream."
      ]
    }

## Child JSON Values {#child_json_values}

Prefixing of the extension identifier is not required of children of a prefixed
JSON object defined by an RDAP extension.

The following example shows this use with a JSON object.

    {
      "rdapConformance": [
        "rdap_level_0",
        "lunarNIC"  
      ],
      "objectClassName": "domain",
      "ldhName": "example.com",
      "remarks":
      [
        {
          "description":
          [
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "lunarNIC_author":
      {
        "firstInitial": "R",
        "lastName": "Heinlein"
      }
    }

Here the JSON name "lunarNIC_author" will separate the JSON from other
extensions that may have an "author" structure. But the JSON contained
within "lunarNIC_author" need not be prepended as the extension collision
is avoided by "lunarNIC_author".

## Object Classes in Extensions {#object_classes_in_extensions}

As described in [@!RFC9082] and (#usage_in_queries), an extension may define new paths in URIs.
If the extension describes the behavior of an RDAP query using that path to return a new RDAP
object class, the JSON names are not required to be prepended with the extension identifier
as described in (#child_json_values). However, the extension MUST define the value for the
`objectClassName` string which is used by clients to evaluate the type of the response., 
To avoid collisions with object classes defined in other extensions, the value for the 
`objectClassName` MUST either be prepended with the extension identifier or
be the extension identifier in cases where the extension defines only one object class.

    {
      "rdapConformance": [
        "rdap_level_0",
        "lunarNIC"  
      ],
      "objectClassName": "lunarNIC author",
      "author":
      {
        "firstInitial": "R",
        "lastName": "Heinlein"
      }
    }

Because the `objectClassName` is a string and [@!RFC9083] sets the precedent of using
spaces in object class names (i.e. "ip network"), extensions may follow the same
convention. It is RECOMMENDED that object class names be lower cased, ASCII characters
that use the space character as a word separator.

## Search Results in Extensions {#search_results_in_extensions}

As described in [@!RFC9082] and (#usage_in_queries), an extension may define new paths in URIs.
If the extension describes the behavior of an RDAP query using the path to return a new RDAP
search result, the JSON name of the search result MUST be prepended with the extension identifier
(to avoid collision with search results defined in other extensions).
If the search result contains object class instances defined by the extension, each instance
must have an `objectClassName` string as defined in (#object_classes_in_extensions).

    {
      "rdapConformance": [
        "rdap_level_0",
        "lunarNIC"  
      ],
      "lunarNIC_authorSearchResult": [
        {
          "objectClassName": "lunarNIC_author",
          "author":
          {
            "firstInitial": "R",
            "lastName": "Heinlein"
          }
        },
        {
          "objectClassName": "lunarNIC_author",
          "author":
          {
            "firstInitial": "J",
            "lastName": "Pournelle"
          }
        },
      ]
    }

## Bare Extension Identifiers {#bare_extension}

Some RDAP extensions define only one JSON value and do not prefix it with their
RDAP extension identifier; instead using the extension identifier as the JSON name
for that JSON value. That is, the extension identifier is used "bare" and not appended
with an underscore character and subsequent names.

Consider the example in (#child_json_values). Using the bare extension identifier pattern,
that example could be written as:

    {
      "rdapConformance": [
        "rdap_level_0",
        "lunarNIC"  
      ],
      "objectClassName": "domain",
      "ldhName": "example.com",
      "remarks":
      [
        {
          "description":
          [
            "She sells sea shells down by the sea shore.",
            "Originally written by Terry Sullivan."
          ]
        }
      ],
      "lunarNIC":
      {
        "firstInitial": "R",
        "lastName": "Heinlein"
      }
    }

Usage of a bare extension identifier contravenes the guidance in [@!RFC9083].
This document updates [@!RFC9083] to explicitly allow this pattern.

# Camel Casing {#camel_casing}

The styling convention used in [@!RFC9083] for JSON names is often called
"camel casing", in reference to the hump of a camel. In this style, the 
first letter of every word, except the first word, composing a name is capitalized.
This convention was adopted to visually separate the namespace from the
name, with an underscore between them.

Though there is no explicit guidance to use camel case names, extensions would
be wise to continue the style.

# Two Classes of Extensions {#extension_classes}

Though all RDAP extensions are to be registered in the IANA RDAP extensions
registry, there is an implicit two-class system of extensions that comes from
the inherit ownership of the RDAP specifications by the IETF: extensions
created by the IETF and extensions not created by the IETF.

In the perspective of how extensions identifiers are used as namespace
separators, extensions created by the IETF are not required to be prefixed
with an extension identifier as the IETF can coordinate its own activities
to avoid name collisions. In practice, most extensions owned by the IETF do use
extension identifiers.

RDAP extensions not defined by the IETF MUST use the extension identifier
as a prefix in accordance with this document, [@!RFC7480], [@!RFC9082], and
[@!RFC9083]. And RDAP extensions defined by the IETF SHOULD use the extension
identifier as a prefix or as a bare extension identifier (see (#bare_extension)).
IETF-defined RDAP extensions that do not follow this guidance MUST describe
the necessity to do so.

# Profile and Marker Extensions {#profiles_and_markers}

Extensions are not required to extend the JSON or URL components of RDAP.

While the RDAP extension mechanism was created to extend RDAP queries
and/or responses, extensions can also be used to signal server policy 
(for example, specifying the conditions of use for existing
response structures). Extensions that are primarily about signaling
server policy are often called "profiles".

Some extensions exist to denote the usage of values placed into an
IANA registry, such as the IANA RDAP registries, or the usage of extensions
to technologies used by RDAP such as extended vCard/jCard properties.
Such extensions exist to "mark" these usages and are often called "marker"
extensions.

For example, an extension may be used to signal desired processing of
a `rel` attribute in a "links" array, where the `rel` value is registered in
the Link Relations Registry (<https://www.iana.org/assignments/link-relations/link-relations.xhtml>).

    {
      "rdapConformance": [
        "rdap_level_0",
        "lunarNIC"  
      ],
      "objectClassName": "domain",
      "ldhName": "example.com",
      "links": [
        {
          "value": "https://example.com/domain/example.com",
          "href": "https://example.com/sideways_href",
          "rel": "sideways",
          "type": "application/rdap+json"
        }
      ]
    }

When defining the usage link relations, extensions should specify the
media types expected to be used with those link relations.

Regardless of the category of these extensions, their usage may also
leverage the appearance of their identifiers in the `rdapConformance` array.
Clients may use the `/help` query as defined in [@!RFC9082] to discover
the extensions available. 

# Extension Versioning

As stated in (#extension_identifier), RDAP extensions are opaque, and
they possess no explicit version despite the fact that some extension
identifiers include trailing numbers. That is, RDAP extensions without
an explicitly defined versioning scheme are opaquely versioned.

For example, `fizzbuzz_1` may be the successor to `fizzbuzz_0`, but it
may also be an extension for a completely separate purpose. Only consultation
of the definition of `fizzbuzz_1` will determine its relationship with
`fizzbuzz_0`. Additionally, `fizzbuzz_99` may be the predecessor of `fizzbuzz_0`.

If a future RFC defines a versioning scheme (such as using the
mechanism defined in (#extension_identifier)), an RDAP extension
definition MUST explicitly denote this compliance.

## Backwards-Compatible Changes {#backwards_compatible_changes}

If an RDAP extension author wants to publish a new version of an
extension that is backwards-compatible with the previous version, then
one option is for the new version of the extension to define a new
identifier, as well as requiring that both the previous identifier
and the new identifier be included in the "rdapConformance" array of 
responses.  That way, clients relying on the previous version of the 
extension will continue to function as intended, while clients wanting 
to make use of the newer version of the extension can check for the new 
identifier in the response.

This approach can be used for an arbitrary number of new
backwards-compatible versions of a given extension.  For an extension
with a large number of backwards-compatible successor versions, this
may lead to a large number of identifiers being included in
responses.  An extension author may consider excluding older
identifiers from the set required by new successor versions,
based on data about client use/support or similar.

## Backwards-Incompatible Changes {#backwards_incompatible_changes}

With the current extension model, an extension with a
backwards-incompatible change is indistinguishable from a new,
unrelated extension.  Implementers of such changes should consider the
following:

 - whether the new version of the extension can be provided alongside
   the old version of the extension, so that a service can simply
   support both during a transition period;
 - whether some sort of client signaling should be supported, so that
   clients can opt for the old or new version of the extension in
   responses that they receive (see
   [@!I-D.newton-regext-rdap-x-media-type] for an example of how this
   might work); and
 - whether the extension itself should define how versioning is
   handled within the extension documentation.

# Extension Identifiers in a Response

Extension specifications have customarily defined only one extension identifier. However,
there is no explicit limit on the number of extension identifiers that may be defined in
a single extension specification. One example is [@?I-D.ietf-regext-rdap-rir-search].

[@!RFC9083, Section 4.1] offers the following guidance on using the extension identifiers
in RDAP responses:

    A response to a "help" request will include identifiers for all of the specifications
    supported by the server. A response to any other request will include only identifiers 
    for the specifications used in the construction of the response.

A strict interpretation of this wording where "construction of the response" only refers
to the JSON syntax would rule out the usage of (#profiles_and_markers) extension IDs which
are in common use in RDAP. This document updates the guidance. For responses to queries
other than "help", a response MUST include in the `rdapConformance` array only those extension
identifiers necessary for a client to deserialize the JSON and understand the semantic meaning
of the content within the JSON, and all extensions identifiers MUST be free from conflict in
both their syntactic and semantic meaning.

Note that this document does not update the guidance from [@!RFC9083, Section 4.1] regarding
"help" responses and the `rdapConformance` array.

When a server implementation supports multiple extensions, it is RECOMMENDED that the server
also support and return versioning information as defined by [@!I-D.gould-regext-rdap-versioning].

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
the ICANN profile or NRO profile extensions, the extension may simply signal 
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
extensions that are similarly non-compliant will not be registered.

To avoid any confusion with the operation of the existing entries, an
extension registration that attempts to use one of the RDAP
conformance values given in this section as an extension identifier
(and so as an RDAP conformance value also) will be rejected.

# Redirects {#redirects}

[@!RFC7480] describes the use of redirects in RDAP. Redirects are prominent
in the discovery of authoritative INRR servers as the process outlined in
[@?RFC9224], which uses IANA allocations, does not account for transfers of
resources between INRRs. [@!RFC7480, Section 4.3] instructs servers to ignore
unknown query parameters. As it relates to issuing URLs for redirects, servers
MUST NOT blindly copy query parameters from a request to a redirect URL as
query parameters may contain sensitive information, such as security credentials,
not relevant to the target server of the URL. Following the advice in [@!RFC7480],
servers SHOULD only place query parameters in redirect URLs when it is known
by the origin server (the server issuing the redirect) that the target server
(the server referenced by the redirect) can process the query parameter and the
contents of the query parameter are appropriate to be received by the target.

As it is unlikely that every server in a cross-authority, redirect scenario will be upgraded to process
every new extension, extensions should not solely rely on query parameters to convey
information about a resource as query parameters are not guaranteed to survive a redirect.

This does not mean extensions are prohibited from using query parameters, but
rather that the use of query parameters must be applied for the scenarios appropriate
for the use of the extension. Therefore, extensions SHOULD NOT rely on query parameters
when the extension is to be used in scenarios requiring clients to find authoritative
servers, such as that described above, or other scenarios using redirects among 
servers of differing authorities.

Extensions MAY use query parameters
in scenarios where the client has a priori knowledge of the authoritative
server to which queries are to be sent. Such scenarios are generally queries
intended to yield multiple results, authentication or authorization,
and other scenarios where the behavior of requests across multiple authorities 
is undefined.

In general, extension authors should be mindful of situations requiring clients
to directly handle redirects at the RDAP layer. Some clients may not be utilizing
HTTP libraries that provide such an option, and many HTTP client libraries that
do provide the option do not provide it as a default behavior. Additionally,
requiring clients to handle redirects at the RDAP layer adds complexity to the
client in that additional logic must be implemented to handle redirect loops,
parameter deconfliction, and URL encoding. The guidance given in [@!RFC7480, section 5.2]
exists to simplify clients, especially those constructed with shell scripts
and HTTP command-line utilities.

# Referrals {#referrals}

It is common in the RDAP ecosystem to link from one RDAP resource to another, 
such as can be found in domain registrations in gTLD DNRs.
These are typically conveyed in the link structure defined in 
[@?RFC9083, section 4.2] and use the "application/rdap+json"
media type.

    {
      "value": "https://regy.example/domain/foo.example",
      "rel": "related",
      "href": "https://regr.example/domain/foo.example",
      "type": "application/rdap+json"
    }

Extensions MUST explicitly define any required behavioral changes to the
processing of referrals, otherwise clients MUST assume the information
provided by referrals requires no additional processing or modification to
use in the dereferencing of the referral.

# Extension Specification Content

The primary purpose of an RDAP extension specification is to aid in
the implementation of RDAP clients. These specifications should consider
the following content guidelines:

1. Examples of RDAP JSON should be generously given, especially in
areas of the specification which may be complex or difficult to describe
with prose.
2. Normative references, i.e. references to materials that are
required for the interoperability of the extension, should be stable
and non-changing.
3. Extension specifications should strongly consider making the use
of HTTPS with RDAP mandatory if appropriate.

# IANA Considerations

[@!RFC7480] defines the RDAP Extensions Registry (<https://www.iana.org/assignments/rdap-extensions/rdap-extensions.xhtml>).
This document does not change the RDAP Extensions Registry nor its purpose. However, this
document does update the procedures to be used by its expert reviewers.

The RDAP Extensions Registry should have as a minimum three expert reviewers
and ideally four or five. An expert reviewer assigned to the review of an RDAP
extension registration must have another expert reviewer double check any
submitted registration.

Expert reviewers are to use the following criteria for extensions defined
in this document, which include but are not limited to:

1. Does the extension define an extension identifier following the naming
conventions described in (#extension_identifier) and (#camel_casing)? For
any recommendations regarding naming conventions (guidance given using
RECOMMENDED, SHOULD, etc.), does the extension describe the need for
departing from the established convention?
2. If the extension defines new queries, does it clearly describe the
expected results of each new query?
3. Does the extension follow the JSON naming as described in (#usage_in_json)?
4. If the extension is a newer version of an older extension, does
the extension specification clearly describe if it is backwards-compatible
(see (#backwards_compatible_changes)) or backwards-incompatible
(see (#backwards_incompatible_changes)).
5. If the extension registers new values in an IANA registry used by RDAP,
does it describe how a client is to use those values?

Extension authors are encouraged but not required to seek an informal review
of their extension by sending a request for review to regext@ietf.org.

# Acknowledgments

The following individuals have provided feedback and contributions to the
content and direction of this document: James Gould, Daniel Keathley, and
Ties de Kock.

{backmatter}
