---
title: "MNA for Stateless MPLS Egress Protection"
abbrev: "SMEP"
category: info

docname: draft-ihle-mpls-mna-stateless-egress-protection-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Multiprotocol Label Switching"
keyword:
 - egress protection
 - resilience
venue:
  group: "Multiprotocol Label Switching"
  type: "Working Group"
  mail: "mpls@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/mpls/"
  github: "uni-tue-kn/mpls-mna-stateless-egress-protection"
  latest: "https://uni-tue-kn.github.io/mpls-mna-stateless-egress-protection/draft-ihle-mpls-mna-stateless-egress-protection.html"

author:
 -
    fullname: Fabian Ihle
    organization: University of Tuebingen
    email: fabian.ihle@uni-tuebingen.de
 -
    fullname: Michael Menth
    organization: University of Tuebingen
    email: michael.menth@uni-tuebingen.de

normative:

informative:


--- abstract

The MPLS Network Action (MNA) framework provides a general mechanism for the encoding and processing of network actions and their data.

The MPLS Egress Protection Framework specifies a fast reroute mechanism for protecting IP/MPLS services.
To that end backup tunnels have to be signaled to the Point of Local Repair (PLR).

This document defines the encoding for the Stateless MPLS Egress Protection (SMEP) network action.
The SMEP network action protects egress routers by providing an alternative MPLS egress label in-stack.
For SMEP, no signaling of protection tunnels is required.


--- middle

# Introduction

MPLS egress protection in {{?RFC8679}} establishes backup tunnels for egress routers on an egress failure, i.e., on a node or a link failure.
This is referred to as egress protection.
The protection mechanism relies on a Point of Local Repair (PLR) that performs local failure detection and local repair.
Usually, this PLR is the penultimate router.
On an egress failure, packets are rerouted using fast reroute (FRR) to an alternative egress router.
The PLR must maintain a list of bypass tunnels and the bypass forwarding state on a per-transport-tunnel basis.
The bypass tunnels are signaled using existing mechanisms, i.e., via an IGP, or topology-driven label distribution protocols such as LDP.

With the MPLS Network Action (MNA) framework, network actions are encoded in the MPLS stack.
{{?I-D.ietf-mpls-mna-hdr}} defines the encoding of such network actions and their data in the MPLS stack.
Those network actions are processed by all nodes on a path (hop-by-hop), by only selected nodes, or on an ingress-to-egress basis.

This document defines the Stateless MPLS Egress Protection (SMEP) network action.
With SMEP, egress bypass tunnels are carried in a network action in the MPLS stack.
The ingress router pushes the MPLS stack containing the SMEP network action.
On an egress failure, the bypass MPLS label in the network action is used to protect the egress tunnel.
No signaling is required for this approach and no state must be kept in the PLR.

## Terminology

This document makes use of the terms defined in {{?RFC8679}} and in {{?I-D.ietf-mpls-mna-hdr}}.

{::boilerplate bcp14-tagged}

# MPLS Network Action for Stateless Egress Protection

## Encoding

~~~~
{::include ./drawings/smep-encoding.txt}
~~~~
{: #fig-smep-encoding title="MNA for Stateless Egress Protection."}

~~~~
{::include ./drawings/smep-encoding_ad.txt}
~~~~
{: #fig-smep-encoding_ad title="MNA for Stateless Egress Protection using a list of bypass labels."}

The network action for stateless MPLS egress protection is encoded as follows:

- Network Action Indication: The SMEP network action is indicated by opcode TBA1.

- Format: The SMEP network action MUST be encoded using a Format C LSE as defined in {{?I-D.ietf-mpls-mna-hdr}}, see {{fig-smep-encoding}}. Optionally, a list of bypass MPLS labels MAY be carried as Format D LSE, see {{fig-smep-encoding_ad}}. In the Format D LSE, the bypass MPLS label is encoded in the least-significant bits of the first data field. The two most-significant bits of the first data field, and the 8 bits of the second data field MUST be set to zero.

- Scope: The SMEP network action is valid only in the select scope.

- Ancillary Data: The SMEP network action requires 20 bits of in-stack ancillary data to encode the bypass MPLS label. The most-significant 16 bit of the bypass MPLS label are located in the first data field of an format C LSE. The least-significant 4 bit are located in the second datafield of an format C LSE. No post-stack data is required.

## Processing

The ingress LER that pushes an SR-MPLS label stack onto a packet includes the bypass MPLS label in a network action.
The bypass MPLS label encodes the SID to an alternative egress router.
The SMEP network action must be placed in the MPLS stack such that the PLR (Point of Local Repair), i.e., the penultimate node, is able to process the network action.
That means the SMEP network action is only processed by the penultimate node using the select scope.
On an egress node failure or an egress link failure the penultimate node MUST use the bypass MPLS label to route traffic to an alternative egress router.
To that end, the data fields in the Format C LSE of the SMEP network actions are concatened to form the 20 bit MPLS label.

# Example

An example topology using MPLS egress protection is shown in {{fig-example1}}.
Labels A and B are used to forward to the penultimate router.
From here, three paths are available to an egress node.
Labels C, C', and C'' are used to route to one of the egress nodes.
If the egress link or router C fails, the PLR can use the bypass tunnel of router C' and C''.
The MPLS stack encoding this functionality is shown in {{fig-example1_stack}}.
The Network Action Sub-Stack (NAS) for SMEP contains an Format A LSE to indicate the MNA Sub-Stack and an Format B LSE.
This is required by {{?I-D.ietf-mpls-mna-hdr}}.
The Format B LSE can contain arbitrary network actions.

~~~~
{::include ./drawings/smep-example1.txt}
~~~~
{: #fig-example1 title="Example network topology with protected egress routers."}

~~~~
{::include ./drawings/smep-example1_stack.txt}
~~~~
{: #fig-example1_stack title="Example MPLS stack for above topology."}

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} apply to this document.


# IANA Considerations

This document requests that IANA allocates a new codepoint with the name "Stateless MPLS Egress Protection" in the "Network Action Opcodes Registry".

| MNA Opcode |  Description                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|    TBA1    |  Stateless MPLS Egress Protection |  This document
{: #table_iana title="SMEP Opcode IANA allocation."}


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
