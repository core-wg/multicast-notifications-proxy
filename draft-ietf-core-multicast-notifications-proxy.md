---
v: 3

title: Using Proxies for Observe Notifications as CoAP Multicast Responses
abbrev: Proxies for Observe Multicast Notifications
docname: draft-ietf-core-multicast-notifications-proxy-latest

ipr: trust200902
area: WIT
wg: CoRE Working Group
kw: Internet-Draft
cat: info
submissiontype: IETF

coding: utf-8

author:
      -
        ins: M. Tiloca
        name: Marco Tiloca
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: marco.tiloca@ri.se
      -
        ins: R. Höglund
        name: Rikard Höglund
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: rikard.hoglund@ri.se
      -
        ins: C. Amsüss
        name: Christian Amsüss
        org:
        street: Hollandstr. 12/4
        city: Vienna
        code: 1020
        country: Austria
        email: christian@amsuess.com
      -
        ins: F. Palombini
        name: Francesca Palombini
        org: Ericsson AB
        street: Torshamnsgatan 23
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: francesca.palombini@ericsson.com

normative:
  I-D.ietf-core-groupcomm-bis:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-ace-key-groupcomm-oscore:
  I-D.ietf-core-href:
  I-D.ietf-core-observe-multicast-notifications:
  RFC4944:
  RFC6838:
  RFC7120:
  RFC7252:
  RFC7641:
  RFC7967:
  RFC8085:
  RFC8126:
  RFC8288:
  RFC8610:
  RFC8613:
  RFC8949:
  RFC9203:
  COSE.Algorithms:
    author:
      org: IANA
    date: false
    title: COSE Algorithms
    target: https://www.iana.org/assignments/cose/cose.xhtml#algorithms
  COSE.Key.Types:
    author:
      org: IANA
    date: false
    title: COSE Key Types
    target: https://www.iana.org/assignments/cose/cose.xhtml#key-type
  COSE.Header.Parameters:
    author:
      org: IANA
    date: false
    title: COSE Header Parameters
    target: https://www.iana.org/assignments/cose/cose.xhtml#header-parameters

informative:
  I-D.ietf-core-coap-pubsub:
  I-D.tiloca-core-oscore-discovery:
  I-D.ietf-core-coral:
  I-D.ietf-core-cacheable-oscore:
  I-D.ietf-cose-cbor-encoded-cert:
  I-D.ietf-core-oscore-capable-proxies:
  I-D.ietf-core-dns-over-coap:
  RFC5280:
  RFC6690:
  RFC7519:
  RFC8392:
  RFC9147:
  RFC9176:
  RFC9200:
  MOBICOM99:
    author:
      -
        ins: S.-Y. Ni
        name: Sze-Yao Ni
      -
        ins: Y.-C. Tseng
        name: Yu-Chee Tseng
      -
        ins: Y.-S. Chen
        name: Yuh-Shyan Chen
      -
        ins: J.-P. Sheu
        name: Jang-Ping Sheu
    title: The Broadcast Storm Problem in a Mobile Ad Hoc Network
    seriesinfo: Proceedings of the 5th annual ACM/IEEE international conference on Mobile computing and networking
    date: 1999-08
    target: https://people.eecs.berkeley.edu/~culler/cs294-f03/papers/bcast-storm.pdf

entity:
  SELF: "[RFC-XXXX]"

--- abstract

The Constrained Application Protocol (CoAP) allows clients to "observe" resources at a server and to receive notifications as unicast responses upon changes of the resource state. Instead of sending a distinct unicast notification to each different client, a server can alternatively send a single notification as a response message over multicast, to all the clients observing the same target resource. When doing so, the security protocol Group Object Security for Constrained RESTful Environments (Group OSCORE) can be used to protect multicast notifications end-to-end between the server and the observer clients. This document describes how multicast notifications can be used in network setups that leverage a proxy, e.g., in order to accommodate clients that are not able to directly listen to multicast traffic.

[^status]

[^status]:
The present version -00 refers to version -12 of draft-ietf-core-observe-multicast-notifications, which includes content about proxies that is also included in the present document. Such content will be removed from draft-ietf-core-observe-multicast-notifications in its next revision.

--- middle

# Introduction # {#intro}

The Constrained Application Protocol (CoAP) {{RFC7252}} has been extended with a number of mechanisms, including resource Observation {{RFC7641}}. This enables CoAP clients to register at a CoAP server as "observers" of a resource, and hence being automatically notified with an unsolicited response upon changes of the resource state.

CoAP supports group communication {{I-D.ietf-core-groupcomm-bis}}, e.g., over IP multicast. This includes support for Observe registration requests over multicast, in order for clients to efficiently register as observers of a resource hosted at multiple servers.

In a number of use cases, it is conversely desirable that a server sends observe notifications for the same target resource to multiple observers at once. In general, this is beneficial when several CoAP clients observe the same target resource at a CoAP server, and thus they could all be notified at once by means of a single response message.

To this end, {{I-D.ietf-core-observe-multicast-notifications}} defines a method that a server can use to deliver observe notifications as CoAP responses addressed to multiple clients, e.g., over IP multicast. Also, it defines how to use the security protocol Group Object Security for Constrained RESTful Environments (Group OSCORE) {{I-D.ietf-core-oscore-groupcomm}} to protect multicast notifications end-to-end between the server and the observer clients.

This document describes how the method specified in {{I-D.ietf-core-observe-multicast-notifications}} can be used in network setups that leverage a proxy, e.g., in order to accommodate clients that are not able to directly listen to multicast traffic.

## Terminology ## {#terminology}
{::boilerplate bcp14-tagged}

Readers are expected to be familiar with terms and concepts described in CoAP {{RFC7252}}, group communication for CoAP {{I-D.ietf-core-groupcomm-bis}}, Observe {{RFC7641}}, Concise Data Definition Language (CDDL) {{RFC8610}}, Concise Binary Object Representation (CBOR) {{RFC8949}}, Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}}, Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}, and Constrained Resource Identifiers (CRIs) {{I-D.ietf-core-href}}.

Readers are also expected to be familiar with terms and concepts described in {{I-D.ietf-core-observe-multicast-notifications}}, particularly with the terms "traditional observation", "group observation", "phantom request", and "informative response".

# TBD

TBD

# IANA Considerations # {#iana}

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

## TBD

TBD

--- back

# TBD

TBD

# Document Updates # {#sec-document-updates}
{:removeinrfc}

## Version -00 ## {#sec-00}

* Imported content about proxies from draft-ietf-core-observe-multicast-notifications-12.

# Acknowledgments # {#acknowldegment}
{: numbered="no"}

The authors sincerely thank {{{Carsten Bormann}}}, {{{Klaus Hartke}}}, {{{Jaime Jiménez}}}, {{{Matthias Kovatsch⁩}}}, {{{John Preuß Mattsson}}}, {{{Jim Schaad}}}, {{{Ludwig Seitz}}}, and {{{Göran Selander}}} for their comments and feedback.

The work on this document has been partly supported by the Sweden's Innovation Agency VINNOVA and the Celtic-Next projects CRITISEC and CYPRESS; and by the H2020 project SIFIS-Home (Grant agreement 952652).
