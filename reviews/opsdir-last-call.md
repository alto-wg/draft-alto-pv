> Thanks for the review and suggestion. We agree that more concrete use cases
> and examples will be helpful and will revise the document accordingly. Please
> see inline for more detailed comments.

I believe the document is generally well written, and the problem space it is
addressing is one for which there is value in defining a solution, but I feel
the document suffers from being too abstract and vague about what it is
defining, and its consideration of practical use cases could be improved.  Thus
I feel at this stage it is Not Ready for publication.

General comments:

The use cases defined are quite varied - large scale analytics, mobile and
CDNs.

> The document was first originated to support the data analytics use case, but
> later was found to be useful in other scenarios. We will focus on the
> analytics use case in the next revision.

SENSE and LHC are not specifically data analytics use cases in the usual
sense of the word, rather SENSE is a model for orchestrating network links (and
capacity) between sites, and the LHC provides large scale data sets for four
major experiments that are distributed and computed upon via the WLCG
(worldwide large hadron collider computing grid).

For LHC, QoE is not so much about time to complete; the important point is not
to have data backlogging if performance drops.

For the WLCG, two networks have evolved over many years to carry the traffic
from the four main experiments; LHCOPN, the optical network, and LHCONE, the
overlay network, both of which are ‘manually’ configured, and with enough
capacity for the traffic thanks to regular network forward look exercises.
While a little complex to administer, other emerging disciplines have expressed
interest in using LHCONE to move data, and some have established agreements
(e.g. SKA, I believe).  While a means to provision capacity on demand would be
attractive, the R&E networks typically have capacity, LHCOPN/LHCONE carry the
LHC traffic, and bottlenecks are in the end sites (hence the evolution of the
Science DMZ principles).

> Thanks very much for the clarification. Indeed we intermingled LHC with other
> data analytics systems, which typically use the coflow abstraction [1] and
> optimize for job completion time. We will clarify in the next revision that
> different analytics systems have different QoE objectives and illustrate how
> the path vector extension can support these use case respectively.

Some specific examples of ANEs would be very helpful.  While the document does
contain examples, they are not grounded around a use case I can readily relate
to, such as the orchestration of a large data flow between two sites in
different R&E networks.  Can the doc show some real examples?

> That is a very good suggestion. We will add more examples in the next
> revision to better motivate the use of ANE.

Section 3 talks of definitions of ANEs being “similar to” Network Elements in
RFC2216, but this is vague.  The topology in Figure 5 is quite simple, as an
example; something more realistic would be interesting.

> We will add a more realistic example to motivate the definition of ANE and the
> initial properties. As figure 5 is used to illustrate the examples of message
> formats, we will move it to the example section.

Ultimately, if ALTO clients have the full network topology even then they may
not know about the routing that occurs by default, so implicitly there's an
assumption of a capability to steer traffic to meet a request.

> This is not true. With path vector, the routing is already specified for a
> given source and destination pair. Thus, the client must not assume that the
> ALTO server has the capability to modify the routing. In fact, for most cases,
> the network only exposes information about the path and does not provide any
> control capability inside the network. For certain use cases, for example, if
> a network allows clients to reserve bandwidth for end-to-end communication, it
> may configure an ALTO server to provide the `max-reservable-bandwidth`
> property. Note this is not an issue specific to the path vector document but
> to the ALTO framework: ALTO carries the semantics of the information but how
> to use the information depends on a higher-layer protocol.

What is the “request” referred to in 5.1.2, for example?

> The requests in 5.1.2 are referring to HTTP requests to ALTO services.

It seems that the document argues that ‘bottlenecks’ are typically capacity
based; do ANEs include specific links, rather than routers, firewalls, etc?   A
stateful firewall can be a significant bottleneck on throughput, for example.

> No, ANE can include routers, firewalls and other middleboxes. However, an ALTO
> server may not want and may not need to distinguish what the bottleneck really
> is -- it is actually one reason why we use the term "abstract network
> element". For example, the maximum throughput of a firewall can be considered
> as the capacity of the ANE exposed to the ALTO clients. We will add the
> firewall example to illustrate the use of ANE in the next revision.

In 4.2.1 it talks of ALTO client identifying bottlenecks; a little more
discussion and examples of that would be useful, for practical use cases such
as an international R&E data transfer.

> We will add more discussions on identifying bottlenecks for this use cases.
> Some pointers are attached below.

The discussion on p.9 about multiple flows is a little odd; in practice in R&E
networks large transfers use tools like GridFTP which uses multiple parallel
TCP flows, such that loss on individual flows does not severely impact
throughput.  Of course, BBR also reduces this concern.

> For GridFTP and BBR, the multiple flows are established between the same
> source and destination but the example contains two "flows" of two source and
> destination pairs. The "multiple flows" in the example, however, represent
> data transfers between different source and destination pairs but of the same
> task (as in the coflow setting [1]).

> Handling multiple flows between the same source and destination pair is
> certainly an important use case. However, it cannot be solved completely by
> the path vector draft alone. We have an individual draft call "flow cost
> service" which can potentially providing information for this use case,
> together with the path vector extension.

Is the use of ALTO designed for single domain, or can it span multiple domains?
 It seems the latter, given the definition of ANE domains, but for the latter
there is no specific model for the common definition of ANEs.

> Existing specifications and extensions are designed for a single
> administrative domain. The term "ANE domain" might be misinterpreted: the
> domain here does not refer to a network domain. Rather, it is inherited from
> the "entity domain" defined in Sec 3.2 in I-D.ietf-alto-unified-prop-new
> document, which is used more in the mathematical sense of "domain": the set of
> valid objects of a specific type. In the unified property extension, an entity
> domain is defined by a specific ALTO resource (called defining information
> resource).

Given the definition of ANEs and PVs, how is traffic then orchestrated or
optimised?  Some pointers here would be useful.  SENSE may be one example.
From my own discussion with people involved with SENSE (and AutoGOLE which uses
it) there is as yet no use of ALTO (rather SENSE uses its own methods to
orchestrate based on intent-based descriptors), but it is something that may be
considered in the future.

> There are different ways to realize the traffic reservation: MPLS tunnels,
> OpenFlow rules, or end-based traffic control (e.g., Linux tc command). For
> specific orchestration mechanisms, please see below ([3]-[5]) for some pointers.

> We will add the pointers to the "Use Cases" section in the next revision.

[1] Chowdhury, M. and Stoica, I. 2012. Coflow: A Networking Abstraction for Cluster
Applications. Proceedings of the 11th ACM Workshop on Hot Topics in Networks
(New York, NY, USA, 2012), 31–36.

[2] https://tools.ietf.org/search/draft-gao-alto-fcs-06

[3] https://datatracker.ietf.org/doc/html/draft-ietf-alto-unified-props-new-18


[4] Viswanathan, R., Ananthanarayanan, G. and Akella, A. 2016. CLARINET:
WAN-Aware Optimization for Analytics Queries. 12th USENIX Symposium on Operating
Systems Design and Implementation (OSDI 16) (Savannah, GA, 2016), 435–450.


[5] Xiang, Q., Chen, S., Gao, K., Newman, H., Taylor, I., Zhang, J. and Yang,
Y.R. 2017. Unicorn: Unified resource orchestration for multi-domain,
geo-distributed data analytics. 2017 IEEE SmartWorld, Ubiquitous Intelligence
Computing, Advanced Trusted Computed, Scalable Computing Communications, Cloud
Big Data Computing, Internet of People and Smart City Innovation
(SmartWorld/SCALCOM/UIC/ATC/CBDCom/IOP/SCI) (Aug. 2017), 1–6.

[6] Xiang, Q., Zhang, J.J., Wang, X.T., Liu, Y.J., Guok, C., Le, F., MacAuley, J.,
Newman, H. and Yang, Y.R. 2018. Fine-grained, Multi-domain Network Resource
Abstraction As a Fundamental Primitive to Enable High-performance, Collaborative
Data Sciences. Proceedings of the International Conference for High Performance
Computing, Networking, Storage, and Analysis (Piscataway, NJ, USA, 2018),
5:1-5:13.


What of non-ALTO traffic on the same links; is the approach to reserve x%
capacity of a link for ALTO orchestrated traffic (the SENSE approach, I
believe)?

> In this case, ALTO is mainly used to expose the capacity information to the
> client. How the resource reservation is actually achieved is not in the scope
> of the document.
