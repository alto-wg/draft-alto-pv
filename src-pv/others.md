# Compatibility {#SecComp}

## Compatibility with Legacy ALTO Clients/Servers

The multipart filtered cost map resource and the multipart endpoint cost
resource has no backward compatibility issue with legacy ALTO clients and
servers. Although these two types of resources reuse the media types defined in
the base ALTO protocol for the accept input parameters, they have different
media types for responses. If the ALTO server provides these two types of
resources, but the ALTO client does not support them, the ALTO client will
ignore the resources without conducting any incompatibility.

<!--
The path vector extension on Filtered Cost Map and Endpoint Cost Service is
backward compatible with the base ALTO protocol:

- If the ALTO server provides extended capabilities `dependent-property-map` and
  `allow-compound-response` for Filtered Cost Map or Endpoint Cost Service, but
  the client only supports the base ALTO protocol, then the client will ignore
  those capabilities without conducting any incompatibility.
- If the client sends a request with the input parameter `properties`, but the
  server only supports the base ALTO protocol, the server will ignore this
  field.
-->

## Compatibility with Multi-Cost Extension ##

<!-- FIXME: path-vector cannot be used in multi-cost, also no reason -->

This document does not specify how to integrate the `path-vector` cost mode with
the multi-cost extension {{RFC8189}}. Although there is no reason why somebody
has to compound the path vectors with other cost types in a single query, there
is no compatible issue doing it without constraint tests.

<!--
As [](#fcm-cap) mentions, the syntax and semantics of whether `constraints` or
`or-constraints` field for the `array` cost mode is not specified in this
document. So if an ALTO server provides a resource with the `array` cost mode
and the capability `cost-constraints` or `testable-cost-types-names`, the ALTO
client MAY ignore the capability `cost-constraints` or
`testable-cost-types-names` unless the implementation or future documents
specify the behavior.
-->

<!--
Cost type path-vector is not a testable cost type. Any format of constraints
SHOULD NOT be applied to cost type path-vector in order for multi-cost to
support the path-vector extension. Specifically,

- Cost type path-vector MUST NOT be included in `testable-cost-types-names` or
  `testable-cost-types`.
- When `testable-cost-types-names` is omitted in the `capabilities` and
  `testable-cost-types` is omitted in the input parameters, `constraints` or
  `or-constraints` SHOULD NOT add any format of constraints on cost type
  path-vector.
-->

## Compatibility with Incremental Update ##

<!-- FIXME: using resource-id header in MIME part -->

The extension specified in this document is NOT compatible with the original
incremental update extension {{I-D.ietf-alto-incr-update-sse}}. A legacy ALTO
client CANNOT recognize the compound client-id, and a legacy ALTO server MAY
use the same client-id for updates of both parts.

ALTO clients and servers MUST follow the specifications given in this document
to ensure compatibility with the incremental update extension.

## Compatibility with Cost Calendar

The extension specified in this document is compatible with the Cost Calendar
extension {{I-D.ietf-alto-cost-calendar}}. When used together with the Cost
Calendar extension, the cost value between a source and a destination is an
array of path vectors, where the k-th path vector refers to the abstract network
paths traversed in the k-th time interval by traffic from the source to the
destination.

When used with time-varying properties, e.g., maximum reservable bandwidth
(maxresbw), a property of a single entity may also have different values in
different time intervals. In this case, an ANE with different property values
MUST be considered as different ANEs.

The two extensions combined together CAN provide the historical network
correlation information for a set of source and destination pairs. A network
broker or client MAY use this information to derive other resource requirements
such as Time-Block-Maximum Bandwidth, Bandwidth-Sliding-Window, and
Time-Bandwidth-Product (TBP) (See {{SENSE}} for details.)

# General Discussions {#SecDisc}

<!--
Cost Calendar is proposed as a useful ALTO extension to provide the historical
cost values for Filtered Cost Map Service and Endpoint Cost Service. Since path
vector is an extension to these services, it SHOULD be compatible with Cost
Calendar extension.

However, the calendar of a path-vector (Endpoint) Cost Map is insufficient for
the application which requires the historical data of routing state information.
The (Endpoint) Cost Map can only provide the changes of the paths. But more
useful information is the history of network element properties which are
recorded in the dependent Network Element Property Map.

Before the Unified Property Map is introduced as an ALTO extension, Filtered
Cost Map Service and Endpoint Cost Service are the only resources which require
the calendar supported. Because other resources don't have to be updated
frequently. But Network Element Property Map as a use case of Unified Property
Map will collect the real-time information of the network. It SHOULD be updated
as soon as possible once the metrics of network elements change.

So the requirement is to provide a general calendar extension which not only
meets the Filtered Cost Map and Endpoint Cost Service but also applies to the
Property Map Service.
-->

## Constraint Tests for General Cost Types ##

The constraint test is a simple approach to query the data. It allows users to
filter the query result by specifying some boolean tests. This approach is
already used in the ALTO protocol. {{RFC7285}} and {{RFC8189}} allow ALTO
clients to specify the `constraints` and `or-constraints` tests to better
filter the result.

However, the current defined syntax is too simple and can only be used to test the
scalar cost value. For more complex cost types, like the `array` mode defined
in this document, it does not work well. It will be helpful to propose more
general constraint tests to better perform the query.

In practice, it is too complex to customize a language for the general-purpose
boolean tests, and can be a duplicated work. So it may be a good idea to
integrate some already defined and widely used query languages (or their
subset) to solve this problem. The candidates can be XQuery and JSONiq.

## General Multipart Resources Query ##

Querying multiple ALTO information resources continuously MAY be a general
requirement. And the coming issues like inefficiency and inconsistency are also
general. There is no standard solving these issues yet. So we need some approach
to make the ALTO client request the compound ALTO information resources in a
single query.

# Security Considerations {#SecSCons}

This document is an extension of the base ALTO protocol, so the Security
Considerations {{RFC7285}} of the base ALTO protocol fully apply when this
extension is provided by an ALTO server.

<!-- Additional security considerations -->

<!-- ## Privacy Concerns { #pricon } -->

The path vector extension requires additional considerations on two security
considerations discussed in the base protocol: confidentiality of ALTO
information (Section 15.3 of {{RFC7285}}) and availability of ALTO service
(Section 15.5 of {{RFC7285}}).

For confidentiality of ALTO information, a network operator should be aware of
that this extension may introduce a new risk: the path vector information may
make network attacks easier. For example, as the path vector information may
reveal more network internal structures than the base protocol, an ALTO client
may detect the bottleneck link and start a distributed denial-of-service (DDoS)
attack involving minimal flows to conduct the in-network congestion.

To mitigate this risk, the ALTO server should consider protection mechanisms to
reduce information exposure or obfuscate the real information, in particular,
in settings where the network and the application do not belong to the same
trust domain. But the implementation of path vector extension involving
reduction or obfuscation should guarantees the constraints on the requested
properties are still accurate.

<!--
On the other hand, in a setting of the same trust domain, a key benefit
of the path-vector abstraction is to reduce information transferred from the network
to the application.
-->

For availability of ALTO service, an ALTO server should be cognizant that using
path vector extension might have a new risk: frequent requesting for path
vectors might conduct intolerable increment of the server-side storage and
break the ALTO server. It is known that the computation of path vectors is
unlikely to be cacheable, in that the results will depend on the particular
requests (e.g., where the flows are distributed). Hence, the service providing
path vectors may become an entry point for denial-of-service attacks on the
availability of an ALTO server. To avoid this risk, authenticity and
authorization of this ALTO service may need to be better protected.

# IANA Considerations # {#SecIANA}

## ALTO Cost Mode Registry ##

This document specifies a new cost mode `path-vector`. However, the base ALTO protocol
does not have a Cost Mode Registry where new cost mode can be registered. This
new cost mode will be registered once the registry is defined either in a
revised version of {{RFC7285}} or in another future extension.

## ALTO Entity Domain Registry ##

As proposed in Section 9.2 of {{I-D.ietf-alto-unified-props-new}}, `ALTO Domain
Entity Registry` is requested. Besides, a new domain is to be registered, listed in
{{tbl-entity-domain}}.


| Identifier | Entity Address Encoding | Hierarchy &amp; Inheritance |
|------------|-------------------------|-----------------------------|
| ane        | See {{entity-address}}  | None                        |
{: #tbl-entity-domain title="ALTO Entity Domain"}

## ALTO Entity Property Type Registry ##

The `ALTO Entity Property Type Registry` is required by the
ALTO Domain `ane`, listed in {{tbl-prop-type-reg}}.

| Identifier              | Intended Semantics                                                   |
|-------------------------|----------------------------------------------------------------------|
| ane:maxresbw            | The maximum reservable bandwidth                                     |
| ane:persistent-entities | An array of identifiers of persistent entities that reside in an ANE |
{: #tbl-prop-type-reg title="ALTO Entity Property Types"}

## ALTO Resource Entity Domain Export Registries ##

### costmap

| Entity Domain Type | Export Function     |
|--------------------|---------------------|
| ane                | See {{costmap-ede}} |
{: title="ALTO Cost Map Entity Domain Export"}

### endpointcost

| Entity Domain Type | Export Function |
|--------------------|-----------------|
| ane                | See {{ec-ede}}  |
{: title="ALTO Endpoint Cost Entity Domain Export"}

# Acknowledgments #

The authors would like to thank discussions with Andreas
Voellmy, Erran Li, Haibin Song, Haizhou Du, Jiayuan Hu, Qiao Xiang, Tianyuan Liu,
Xiao Shi, Xin Wang, and Yan Luo. The authors thank Greg Bernstein (Grotto Networks),
Dawn Chen (Tongji University), Wendy Roome, and Michael Scharf for
their contributions to earlier drafts.
