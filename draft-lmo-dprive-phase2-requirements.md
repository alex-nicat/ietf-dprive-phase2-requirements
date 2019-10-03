---
title: DNS Privacy Requirements for Exchanges between Recursive Resolvers and Authoritative Servers
abbrev: DPRIVE Phase 2 Requirements
docname: draft-lmo-dprive-phase2-requirements-00
category: info

ipr: trust200902
area: General
workgroup: DPRIVE
keyword: DNS, Encryption, Privacy 

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: J. Livingood
    name: Jason Livingood
    organization: Comcast
    email: Jason_Livingood@comcast.com
 -
    ins: A. Mayrhofer
    name: Alexander Mayrhofer
    organization: nic.at GmbH
    email: alex.mayrhofer.ietf@gmail.com
 - 
    ins: B. Overeinder
    name: Benno Overeinder
    organization: NLnetLabs
    email: benno@NLnetLabs.nl

normative:

informative:

--- abstract

This document provides requirements for adding confidentiality to DNS exchanges between recursive resolvers and authoritative servers. 

--- middle

# Introduction & Scope

The 2018 approved [charter of the IETF dprive working group](https://datatracker.ietf.org/doc/charter-ietf-dprive/) contains milestones related to confidentiality aspects of transactions on the recursor-to-authoritative leg of the DNS ecosystem. One of the work items in that charter is described as:

> Develop requirements for adding confidentiality to DNS exchanges between recursive resolvers and authoritative servers
> (unpublished document).

This is also reflected in the [DPRIVE milestones](https://datatracker.ietf.org/wg/dprive/about/), which (as of Dec 2018) contains a respective milestone:

> Unpublished document on requirements for DNS privacy services between recursive and authoritative servers (Wiki)

In a charter revision of the DPRIVE working group approved in March 2019, the milestone above was upgrade to "unpublished document", with the "wiki" reference removed. This document intends to cover that milestone.

TODO Introduction

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Existing and Previous Work

Even though the previous charter of dprive did consider the recursor-to-authoritative side as out of scope, the issue has been discussed in previous work, including:

- {{?RFC7626}}: The introduction section discusses the information exposed to authoritative servers, and the "Risks" section (section 2) includes discussion of aspects of the recursor-authoritative leg as well.  Section 2.5.2 deals specifically with authoritative servers.

- **TODO** Mailing list pointers (not covered in RFC 7626)

- **Gap analysis**: This seperates into various "areas". From the "pure" protocol level, with the definition of DoT, the "core" transport protocol seems to be done with {{?RFC7858}}. In terms of authentication, the existing "profiles" document {{?RFC8310}} is not sufficient for the auth-to-recursor case - the problems boil down to "bootstrapping". TODO: manu draft? PKI fingerprint in nameserver names? Furthermore, operational concerns might be completely different for the m:n case vs. the n:few case for stub to recurors, though many of the required "session management" specifiation is already there. Privacy problems around TLS session resumption etc. are also not directly applicable to the recursor->auth use case. Security: The properties for recursor->auth are very different from stub->recursive, especially if considering "high-profile" / "critical" auth servers (TLDs? root servers?) 
- **Design Space**: Core protocol:While there may be other options (such as application level encryption of the DNS packets themselves), it appears thatperusing the well known "let's wrap the protocol into an encryption transport layer" model (see HTTP, SMTP, et.al.) is the most efficient way of addressing the problem. However, for sake of completeness, the following would also be viable options:
  - [ConfidentialDNS](https://tools.ietf.org/html/draft-wijngaards-dnsop-confidentialdns-03), other proposals
  - non-IETF work, e.g. (https://dnscurve.org/)
  For authentication / discovery, the design space is much broader, and might require more creative solutions (though, here as well, existing protocols could serve as "templates").

# Perspectives and Use Cases

The DNS resolving process involves several entities.  These entities have different interests/requirements, and hence it does make sense to examine the interests of those entities separately (which doesn’t mean the interests would overlap!).  Three different entities can be identified, and their interests are described in the following sections:

- Users
- Operators
- Implementors / Software Vendors

Furthermore, the DNS is subject to many studies that are performed by Researchers. It is to be discussed whether the perspective of those entities shall be considered as an additional perspective.

## The User Perspective and Use Cases

The privacy and confidentiality of Users (that is, users as in clients of recursive resolvers, which in turn forward/resolve the user's DNS requests by contacting authoritative servers) can be improved in several ways.  We call this "minimisation of exposure", and there are currently three ways to reduce that exposure:

  * Qname minimisation, reducing the amount of information which is absolutely necessary to resolve a query
  * Aggressive NSEC/local auth cache, reducing the amount of outgoing queries in the first place
  * Encryption, removing exposure of information while in transit 

As recursors typically forwards queries received from the user to authoritative servers.  This creates a transitive trust between the user and the recursor, as well as the authoritive server, since information created by the user is exposed to the authoritative server.  However, the user has never a chance to identify which data was exposed to which authoritative party (via which path).

Also, Users would want to be informed about the status of the connections which were made on their behalf, which adds a fourth point

Encryption/privacy status signaling

**TODO**: Actual requirements - what do users "want"? Start below:

## The Operator Perspective and Use Cases

Operators of authoritative services have to provide stable and fast DNS services, and interact with a wide range of clients, not all of them authoritative servers. The operator side actually consists of two sides:

  * The "upstream" facing side of recursive resolvers
  * The "downstream" side of authoritative servers

Those two sides are typically operated by different entities, but many entities operate "both sides".  Even though that is discouraged (**TODO** source), the two sides might even be operated on the same nameserver.

  * Maybe different technical perspectives for operators
    * Intelligence (sharing information)
    * SLD popularity for marketing
  * Focus initially on Second Level Domains (SLDs) initially
    * Is there a difference for TLDs vs. SLDs from a "protocol" perspective?
  * Monitoring and aggregated data analysis
  * Signaling provisioning information
    * New record type for finding authoritative server key and authentication?  Use SRV?  (Being able to use different servers for serving up DNS-over-{TCP,UDP} vs DNS-over-TLS responses may be valuable.
    * Signal secure transport details (DNS-over-TLS, DNS-over-QUIC, EncryptedSNI, connectionless, etc.), perhaps in an extensible manner?  Minimize RTTs and reduce need for trials.
    * Large provider use cases where the NS names are out of bailiwick for the zone (e.g. small number of distinct NS records serving 100k+ zones)
  * EDNS client subnet
  * Decide between TLS and connectionless (such as COSE-based messages)
  * Costs of TLS connection vs. connectionless
    * Technical solution, e.g. encryption of the DNS query, shouldn't enable an attack vector for DDoS or resource exhaustion. For example, only if the client uses DNS-over-TLS, the upstream query to the authoritative will be over DNS-over-TLS also.  If the client uses UDP, the resolver won't invest resources in DNS-over-TLS to prevent a potential resource exhaustion attack.
    * Reuse connection state (if any) and examine resumption considerations
    * Minimize server-side state (eg, with session tickets)
    * Need empirical studies on capacity, traffic, attack vectors
    * Evaluate impact on architecture and footprint expansion
    * Analyze optimal persistent connection time/time-out
    * Analyze optimal number of persistent connections recursive resolvers should maintain
    * Consider operational concerns with respect to capabilities signaling
    * Develop a profile that has operational advantages for operators

**TODO**: Actual requirements - what do operators “want”?

## The Implementor / Software Vendor Perspective and Use Cases

Implementer requirements follows requirements from user and operator perspectives:

  * Non-functional requirements, e.g. diversity of implementations
  * Horizontal vs. vertical scaling, for example similar to http servers
  * Use of DANE for authentication: strict vs. opportunistic
  * Incremental deployment
  * Cache reuse vs. downgrade?  Does the cache need to be partitioned?  When can an in-cache answer retrieved via cleartext be served encrypted to a recursive query?
  * (Use of TCP fast open) 

**TODO**: Actual requirements of implementors - essentially, they follow what Operators need?

# Functional Breakdown of Recursive-to-Authoritative Privacy Aspects

The goal of this section is to describe avenues of work (specific
functionality) that might be neatly separable from one another on the
path toward making these protections widely available.  There are
probably some interdependencies here, but breaking a problem down into
smaller sub-tasks is often a useful way to identify specific tradeoffs
that can be made independently.  The initial breakdown here was
provided by [dkg](mailto:dkg@fifthhorseman.net), but hopefully other
people will contribute to it.

## Privacy Protection Mechanism

How specifically should requests and responses between recursors and
authoritative servers be protected? This might be as simple as "use DNS-over-TLS"

## Authentication

How should clients contacting authoritative servers authenticate the
servers?  How should non-authenticated connections be treated?

## Performance and Efficiency

 * Can authoritative server operators limit resource-exhaustion attacks against private DNS mechanisms from having an impact on traditional (non-private) authoritative DNS availability?
 * What are best practices for authoritative server operators that can minimize latency and unavailability?
 * What are best practices for recursors?

## Detection of availability

Most recursors speak to multiple authoritative nameservers.  Not every
authorititative nameserver will support this mechanism.  How should a
recursor learn of the mechanism's availability? (There may be multiple
ways, or none)

What scope/granularity should such an availability marker have?

 * by zone ("all authoritative nameservers in the `example.net` zone
   support private queries from resolvers")
 * by identified nameserver ("the nameserver `a.ns.example.net`
   supports private queries from resolvers")
 * by IP address ("any namservers that resolve to 192.0.2.13 support
   private queries from resolvers")

Note that if there is no signal for availability, recursors could
still opportunistically try the DNS privacy mechanism.

Should a signal of availability also indicate a preference for privacy
over availability? i.e., are there distinct ways to signal
"DNS-privacy is available" separately from "Only contact this server via
DNS-privacy if you understand this signal (though we may continue to
support non-private DNS queries for clients that don't understand
it)".

## End-user policy propagation

Like any multi-party protocol (e.g. SMTP), the end user's preferences
or policies might or might not be respected by later hops in the
chain.  But if we have a way to express those preferences, we offer
cooperating resolvers at least an opportunity to respect them.

What sorts of preferences or policy might an end-user want to express?
for example:

 * do not identify me (e.g. don't send
   [ECS](https://tools.ietf.org/html/rfc7871) data about me when talking to authoritative servers)
 * prefer DNS privacy over reduced latency (i.e., do not try to do speedups -- try opportunistic privacy first and fall back to cleartext only if that fails)
 * never do non-private authoritative queries on my behalf (for any external queries you need to do to resolve this request, require strict, well-authenticated DNS privacy) 

How specifically are these preferences be expressed by the client?
(e.g. new EDNS0 options?)  Should the recursor have a way to indicate
whether:

 * they are capable of honoring them?
 * they intend to honor them?
 * they *did* honor them over the course of a specific lookup?

If a resolver merely forwards a request to another recursor, should it
also propagate those preferences/policy?  if so, how?

This smells similar to {{?I-D.ietf-uta-smtp-require-tls}}.

# Security Considerations

TODO

# IANA Considerations

This document has no actions for IANA.

--- back

# Acknowledgments
{:numbered="false"}

TODO
