---
title: DNS Privacy Requirements for Exchanges between Recursive Resolvers and Authoritative Servers
abbrev: DPRIVE Phase 2 Requirements
docname: draft-ietf-dprive-phase2-requirements-02
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
    organization: NLnet Labs
    email: benno@NLnetLabs.nl

normative:

informative:

--- abstract

This document describes requirements for adding confidentiality to DNS exchanges between recursive resolvers and authoritative servers. 

--- middle

# Introduction & Scope

The 2018 approved [charter of the IETF DPRIVE Working Group](https://datatracker.ietf.org/doc/charter-ietf-dprive/) contains milestones related to confidentiality aspects of DNS transactions between the iterative resolver and authoritative name servers.

This is also reflected in the [DPRIVE milestones](https://datatracker.ietf.org/wg/dprive/about/), which (as of October 2019) contains two relevant milestones:

>Develop requirements for adding confidentiality to DNS exchanges
between recursive resolvers and authoritative servers (unpublished
document).

>Investigate potential solutions for adding confidentiality to DNS
exchanges involving authoritative servers (Experimental).

This document intends to cover the first milestone for defining requirements for adding confidentiality to DNS exchanges
between recursive resolvers and authoritative servers. This may in turn lead to progress in investigating, developing and standardizing potential experimental methods of meeting those requirements.

The motivation for this work is to extend the confidentiality methods used between a user's stub resolver and a recursive resolver to the recursive queries sent by recursive resolvers in response to a DNS lookup (when a cache miss occurs and the server must perform recursion to obtain a response to the query). A recursive resolver will send queries to root servers, to Top Level Domain (TLD) servers, to authoritative second level domain servers and potentially to other authoritative DNS servers and each of these query/response transactions presents an opportunity to extend the confidentiality of user DNS queries. 

# Document Work Via GitHub

The authors are working on this document via GitHub at https://github.com/alex-nicat/ietf-dprive-phase2-requirements. Feedback via pull requests and issues are invited there. 

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

This document also makes use of DNS Terminology defined in {{!RFC8499}}

# Threat Model and Problem Statement

Currently, protocols such as DoT provide encryption between the user's stub resolver and a recursive resolver. This potentially provides (1) protection from observation of end user DNS queries and responses, (2) protection from on-the-wire modification DNS queries or responses (including potentially forcing a downgrade to an unencrypted communication). Of course, observation and modification are still possible when performed by the recursive resolver, which decrypts queries, serves a response from cache or performs recursion to obtain a response (or synthesizes a response), and then encrypts the response and sends it back to the user's stub resolver. 

But observation and modification threats still exist when a recursive resolver must perform DNS recursion, from the root to TLD to authoritative servers. This document specifies requirements for filling those gaps. 

# Features to Provide Confidentiality

Confidentialty can be provided using a combination of techniques. This section describes the protocol implementation requirements and optional features that can be used to provide confidentiality.

## Requirements

1. Each implementing party MUST be able to independently take incremental steps to meet requirements without the need for close coordination (e.g. loosely coupled)
2. A recursive resolver that supports recursive-to-authoritative DNS encryption MUST be able to determine whether or not a given authoritative name server to which it intends to connect also supports recursive-to-authoritative DNS encryption.
3. An authoritative name server that supports recursive-to-authoritative DNS encryption MUST be able to indicate that it supports recursive-to-authoritative DNS encryption in a way that facilitates (2).
4. An authoritative name server that does not support recursive-to-authoritative MUST NOT have to make any changes to facilitate (2).
5. The secure transport MUST only be established when referential integrity can be verified, MUST NOT have circular dependencies, and MUST be easily analyzed for diagnostic purposes.
[SAH note: I'm not sure what "easily analyzed for diagnostic purposes" means. Could you rephrase this, remove it, or otherwise be clear about how a relative term like "easily" can be a MUST?]
6. Each implementing party MUST be able to negotiate use of a secure transport protocol or other DNS privacy protections in a manner that enables operators to perform appropriate performance and security monitoring, conduct relevant research, etc.
7. The authoritative domain owner or their administrator MUST have the option to specify their secure transport preferences (e.g.  what specific protocols are supported).  This SHALL include a method to publish a list of secure transport protocols (e.g.  DoH, DoT and other future protocols not yet developed).  In addition this SHALL include whether a secure transport protocol MUST always be used (non-downgradable) or whether a secure transport protocol MAY be used on an opportunistic (not strict) basis in recognition that some servers for a domain might use a secure transport protocol and others might not.
8. The authoritative domain owner or their administrator MUST have the option to vary their preferences on an authoritative nameserver to nameserver basis, due to the fact that administration of a particular DNS zone may be delegated to multiple parties (such as several CDNs), each of which may have different technical capabilities.
9. A given name server may be authoritative for multiple zones. As such, a name server MAY support use of a secure transport protocol for one zone, but not for another.
10. The specification of secure transport preferences MUST be performed using the DNS and MUST NOT depend on non-DNS protocols.
11. For secure transports using TLS, TLS 1.3 (or later versions) MUST be supported and downgrades from TLS 1.3 to prior versions MUST not occur.

## Optional Features
1. QNAME minimisation SHOULD be implemented in all steps of recursion 
2. DNSSEC validation SHOULD be performed
3. If an authoritative domain owner or their administrator indicates that (1) multiple secure transport protocols are available, or that (2) a secure transport and insecure transport are available, or that (3) no secure transport is available, then a recursive server SHOULD negotiate selection of an available transport protocol.

# Security Considerations

Authoritative name servers will need to perform additional processing steps, such as completing key exchanges and maintaining persistent connections, when responding to queries from a recursive resolver that requests use of a secure transport protocol. These additional processing steps can have an impact on server availability if they are abused. As such, negotiation and use of a secure transport protocol should be done in a manner that does not increase the risk of an authoritative name server outage or lead a recursive server to fail to communicate with an authoritative name server.

# IANA Considerations

This document has no actions for IANA.

# Changelog

Version 00: Updated prior individual draft following IETF-106 feedback
Version 01: Small editorial changes
Version 02: Incorporate feedback and suggestions from Scott Hollenbeck

# APPENDIX: Perspectives and Use Cases

The DNS resolving process involves several entities.  These entities have different interests/requirements, and hence it does make sense to examine the interests of those entities separately - though in many cases their interests are aligned.  Four different entities can be identified, and their interests are described in the following sections:

- Users
- Operators
- Implementors / Software Developers
- Researchers

## The User Perspective and Use Cases

The privacy and confidentiality of Users (that is, users as in clients of recursive resolvers, which in turn forward/resolve the user's DNS requests by contacting authoritative servers) can be improved in several ways.  We call this "minimisation of exposure", and there are currently three ways to reduce that exposure:

  * Qname minimisation {{?RFC7816}}, reducing the amount of information to what is absolutely necessary to resolve a query
  * Aggressive NSEC/local auth cache {{?RFC8198}}, reducing the amount of outgoing queries in the first place
  * Encryption, removing exposure of information while in transit 

As recursors typically forwards queries received from the user to authoritative servers.  This creates a transitive trust between the user and the recursor, as well as the authoritative server, since information created by the user is exposed to the authoritative server.  However, the user never has a chance to identify what data was exposed to which authoritative party (via which path).

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
  * EDNS client subnet (JL: Not sure ECS crosses the cost/benefit threshold to be included as a requirement and many CDNs that run auth servers will likely say ECS is quite operationally important)
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
  * Use of DANE {{?RFC6698}} for authentication: strict vs. opportunistic
  * Incremental deployment
  * Cache reuse vs. downgrade?  Does the cache need to be partitioned?  When can an in-cache answer retrieved via cleartext be served encrypted to a recursive query?
  * (Use of TCP fast open) - but this might be a requirement for the actual encryption protocol

**TODO**: Actual requirements of implementors - essentially, they follow what Operators need?

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Scott Hollenbeck for his early feedback and providing text for the Internet Draft.
