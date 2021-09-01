# Specification: Basic Data Types {#Basic}

## ANE Name {#ane-name-spec}

An ANE Name is encoded as a JSON string with the same format as that of the type
PIDName (Section 10.1 of {{RFC7285}}).

The type ANEName is used in this document to indicate a string of this
format.

## ANE Domain {#ane-domain-spec}

The ANE domain associates property values with the Abstract Network Elements in
a Property Map. Accordingly, the ANE domain always depends on a Property Map.

### Entity Domain Type ## {#domain-type}

ane

### Domain-Specific Entity Identifier ## {#entity-address}

The entity identifiers are the ANE Names in the associated Property Map.

### Hierarchy and Inheritance

There is no hierarchy or inheritance for properties associated with ANEs.

### Media Type of Defining Resource {#domain-defining}

When resource specific domains are defined with entities of domain type `ane`,
the defining resource for entity domain type `pid` MUST be a Property Map. The
media type of defining resources for the `ane` domain is:

    application/alto-propmap+json

Specifically, for ephemeral ANEs that appear in a Path Vector response, their
entity domain names MUST be exactly ".ane" and the defining resource of these
ANEs is the Property Map part of the multipart response. Meanwhile, for
persistent ANEs whose entity domain name has the format of "PROPMAP.ane" where
PROPMAP is the name of a Property Map resource, PROPMAP is the defining resource
of these ANEs. Persistent entities are `persistent` because standalone queries
can be made by an ALTO client to their defining resources when the connection to
the Path Vector service is closed.

For example, the defining resource of an ephemeral ANE whose entity identifier
is ".ane:NET1" is the Property Map part that contains this identifier. The
defining resource of a persistent ANE whose entity identifier is
"dc-props.ane:DC1" is the Property Map with the resource ID "dc-props".


## ANE Property Name {#ane-prop-name-spec}

An ANE Property Name is encoded as a JSON string with the same format as that of
Entity Property Name (Section 5.2.2 of {{I-D.ietf-alto-unified-props-new}}).

## Initial ANE Property Types

In this document, two initial ANE property types are specified,
`max-reservable-bandwidth` and `persistent-entity-id`.

Note that the two property types defined in this document do not depend on any
information resource, so their ResourceID part must be empty.

### Maximum Reservable Bandwidth {#maxresbw}

The maximum reservable bandwidth property (`max-reservable-bandwidth`) stands
for the maximum bandwidth that can be reserved for all the traffic that
traverses an ANE. The value MUST be encoded as a non-negative numerical cost
value as defined in Section 6.1.2.1 of {{RFC7285}} and the unit is bit per
second (bps). If this property is requested but not present in an ANE, it MUST
be interpreted as that the ANE does not support bandwidth reservation.

### Persistent Entity ID {#persistent-entity-id}

The persistent entity ID property is the entity identifier of the persistent
ANE which an ephemeral ANE presents (See {{assoc}} for details). The value of
this property is encoded with the format EntityID defined in Section 5.1.3 of
{{I-D.ietf-alto-unified-props-new}}.

In this format, the entity ID combines:

- a defining information resource for the ANE on which a
  "persistent-entity-id" is queried, which is the Property Map resource
  defining the ANE as a persistent entity, together with the properties;

- the persistent name of the ANE in that Property Map.

With this format, the client has all the needed information for further
standalone query properties on the persistent ANE.

### Examples

~~~~~~~~~~ drawing
                                     ----- L1
                                    /
        PID1   +----------+ 10 Gbps +----------+    PID3
 192.0.2.0/28+-+ +------+ +---------+          +--+192.0.2.32/28
               | | MEC1 | |         |          |   2001:DB8::3:0/16
               | +------+ |   +-----+          |
        PID2   |          |   |     +----------+
192.0.2.16/28+-+          |   |         NET3
               |          |   | 15 Gbps
               |          |   |        \
               +----------+   |         -------- L2
                   NET1       |
                            +----------+
                            | +------+ |   PID4
                            | | MEC2 | +--+192.0.2.48/28
                            | +------+ |   2001:DB8::4:0/16
                            +----------+
                                NET2
~~~~~~~~~~
{: #fig-pe artwork-align="center" title="Examples of ANE Properties"}

In this document, {{fig-pe}} is used to illustrate the use of the two initial
ANE property types. There are 3 sub-networks (NET1, NET2 and NET3) and two
interconnection links (L1 and L2). It is assumed that each sub-network has
sufficiently large bandwidth to be reserved.

To illustrate the use of `max-reservable-bandwidth`, an ALTO server can create
an ANE for each interconnection link, where the initial value for
`max-reservable-bandwidth` is the link capacity.

To illustrate the use of `persistent-entity-id`, assume the ALTO server has a
Property Map resource called "mec-props" that defines persistent ANEs "MEC1" and
"MEC2" that represent the corresponding mobile edge computing (MEC) clusters.
Since MEC1 is associated with NET1, the `persistent-entity-id` of the ephemeral
ANE `.ane:NET1` is the persistent entity id `mec-props.ane:MEC1`.

## Path Vector Cost Type {#cost-type-spec}

This document defines a new cost type, which is referred to as the `Path Vector`
cost type. An ALTO server MUST offer this cost type if it supports the extension
defined in this document.

### Cost Metric: ane-path {#metric-spec}

The cost metric "ane-path" indicates the value of such a cost type conveys an
array of ANE names, where each ANE name uniquely represents an ANE traversed by
traffic from a source to a destination.

An ALTO client MUST interpret the Path Vector as if the traffic between a source
and a destination logically traverses the ANEs in the same order as they appear
in the Path Vector.

### Cost Mode: array {#mode-spec}

The cost mode "array" indicates that every cost value in the response body of a
(Filtered) Cost Map or an Endpoint Cost Service MUST be interpreted as a JSON
array object.

Note that this cost mode only requires the cost value to be a JSON array of
JSONValue. However, an ALTO server that enables this extension MUST return a
JSON array of ANEName ({{ane-name-spec}}) when the cost metric is
"ane-path".

## Part Resource ID and Part Content ID {#part-rid-spec}

A Part Resource ID is encoded as a JSON string with the same format as that of the
type ResourceID (Section 10.2 of {{RFC7285}}).

Even though the client-id assigned to a Path Vector request and the Part
Resource ID MAY contain up to 64 characters by their own definition, their
concatenation (see {{ref-partmsg-design}}) MUST also conform to the same length
constraint. The same requirement applies to the resource ID of the Path Vector
resource, too. Thus, it is RECOMMENDED to limit the length of resource ID and
client ID related to a Path Vector resource to 31 characters.

A Part Content ID conforms to the format of msg-id as specified in
{{RFC2387}} and {{RFC5322}}. Specifically, it has the following format:

  "<" PART-RESOURCE-ID "@" DOMAIN-NAME ">"

PART-RESOURCE-ID:
: PART-RESOURCE-ID has the same format as the Part Resource ID. It is used to
  identify whether a part message is a Path Vector or a Property Map.

DOMAIN-NAME:
: DOMAIN-NAME has the same format as dot-atom-text specified in Section 3.2.3 of
  {{RFC5322}}. It must be the domain name of the ALTO server.
