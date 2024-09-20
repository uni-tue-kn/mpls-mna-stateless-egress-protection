---
title: "Stateless MNA-based Egress Protection (SMEP)"
abbrev: "SMEP"
category: std

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
    city: Tuebingen
    country: Germany
    email: fabian.ihle@uni-tuebingen.de
 -
    fullname: Michael Menth
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: michael.menth@uni-tuebingen.de

normative:

informative:


--- abstract

The MPLS Network Action (MNA) framework provides a general mechanism for the encoding and processing of network actions and their data.

The MPLS Egress Protection Framework specifies a fast reroute framework for protecting IP/MPLS services.
To that end bypass tunnels have to be signaled to the Point of Local Repair (PLR).
Further, the PLR must maintain the bypass forwarding state.

This document defines the encoding for the Stateless MNA-based Egress Protection (SMEP) network action.
The SMEP network action protects egress routers by providing an alternative MPLS egress label in-stack.
SMEP does not require a bypass forwarding state in PLRs.

--- middle

# Introduction

MPLS egress protection in {{?RFC8679}} establishes bypass tunnels for egress routers on an egress failure, i.e., on a node or a link failure.
This is referred to as egress protection.
The protection mechanism relies on a Point of Local Repair (PLR) to perform local failure detection and local repair.
Typically, this PLR is the penultimate router.
When an egress failure occurs, packets are rerouted to an alternative egress router.
The PLR must maintain a list of bypass tunnels and the bypass forwarding state on a per-transport-tunnel basis.
Typically, this is done using context label switching.
The PLR node maintains the bypass forwarding state, which is a mapping of context labels to bypass tunnels.
The bypass tunnels are signaled using existing mechanisms, i.e., via an IGP, or topology-driven label distribution protocols such as LDP.

With the MPLS Network Action (MNA) framework, network actions are encoded in the MPLS stack.
{{?I-D.ietf-mpls-mna-hdr}} defines the encoding of such network actions and their data in the MPLS stack.
These network actions are processed by all nodes on a path (hop-by-hop), by only selected nodes, or on an ingress-to-egress basis.

This document defines the Stateless MNA-based Egress Protection (SMEP) network action.
With SMEP, egress bypass tunnels are carried in a network action in the MPLS stack.
The egress bypass tunnel is indicated by an alternative MPLS forwarding label in-stack.
This is called the BML (BML).
The ingress router pushes the MPLS stack containing the SMEP network action.
On an egress failure, the BML in the network action is used to protect the egress tunnel.
The PLR node is required to install the MPLS forwarding entries for the bypass tunnels using the BML.
Beside that, no additional signaling is required for this approach and no state needs to be maintained in the PLR.

## Terminology

{::boilerplate bcp14-tagged}

### Abbreviations
This document makes use of the terms defined in {{?RFC8679}} and in {{?I-D.ietf-mpls-mna-hdr}}.

Further abbreviations used in this document:

| Abbreviation |  Meaning                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|    BML    |  BML |  This document
|    SMEP    |  Stateless MNA-based Egress Protection |  This document
{: #table_abbrev title="Abbreviations."}


# MPLS Network Action for Stateless Egress Protection

In this section, we describe the encoding of SMEP and the processing of SMEP in an LSR.

## Encoding

~~~~
{::include ./drawings/smep-encoding.txt}
~~~~
{: #fig-smep-encoding title="MNA for Stateless Egress Protection."}

~~~~
{::include ./drawings/smep-encoding_ad.txt}
~~~~
{: #fig-smep-encoding_ad title="MNA for Stateless Egress Protection using a list of bypass labels."}

The network action for stateless MNA-based egress protection is encoded as follows:

- Network Action Indication: The SMEP network action is indicated by opcode TBA1.

- Format: The SMEP network action MUST be encoded using a Format C LSE as defined in {{?I-D.ietf-mpls-mna-hdr}}, see {{fig-smep-encoding}}. Optionally, a list of BMLs MAY be carried as Format D LSE, see {{fig-smep-encoding_ad}}.

- Scope: The SMEP network action is only valid in the select scope.

- Ancillary Data: The SMEP network action requires 20 bits of in-stack ancillary data to encode the BML. The most-significant 16 bit of the BML are located in the first data field of an Format C LSE. The least-significant 4 bit are located in the second datafield of an Format C LSE. If Format D LSEs are provided, the BML is encoded in the least-significant bits of the first data field of an Format D LSE. The two most-significant bits of the first data field, and the 8 bits of the second data field MUST be set to zero. No post-stack data is required.

## Processing

The ingress LER which pushes an MPLS label stack onto a packet includes the BML in a network action.
The BML encodes the bypass tunnel to an alternative egress router.
The SMEP network action MUST be placed in the MPLS stack such that the PLR (Point of Local Repair), i.e., the penultimate node, is able to process the network action.
This means that the SMEP network action is only processed by the penultimate node using the select scope.
On an egress node failure or an egress link failure, the penultimate node MUST use the BML to route traffic to an alternative egress router.
To that end, the PLR pushes the BML from the Format C and D LSEs to the MPLS stack and pops the incoming label.
A list of BMLs MAY be provided as Format D LSEs to encode a bypass tunnel constructed by Segment Routing.

# Example

A simple example topology using MNA-based egress protection with an SR bypass tunnel is shown in {{fig-example1}}.
Labels A and B are used to forward to the penultimate router.
From here, two paths are available to an egress node.
Label C is used to route to the egress node, and labels C' and C'' are used to route to the backup egress node.
If the egress link or router C fails, the PLR can use the bypass tunnel of router C' and C''.
The MPLS stack pushed by the ingress LER that encodes this functionality for the example topology is shown in {{fig-example1_stack}}.
The Network Action Sub-Stack (NAS) for SMEP contains an Format A LSE to indicate the MNA sub-Stack and an Format B LSE.
This is required by {{?I-D.ietf-mpls-mna-hdr}}.
The Format B LSE can contain arbitrary network actions.

In the example, LSR A and B pop the labels A and B.
On an egress failure, the PLR pops the incoming label C, and the NAS, and pushes the list of BMLs onto the stack.
The label stack after SMEP is applied is shown in {{fig-example1_stack2}}.

~~~~
{::include ./drawings/smep-example1.txt}
~~~~
{: #fig-example1 title="Example network topology with protected egress routers."}

~~~~
{::include ./drawings/smep-example1_stack.txt}
~~~~
{: #fig-example1_stack title="Example MPLS stack pushed by the ingress LER for above topology."}

~~~~
{::include ./drawings/smep-example1_stack2.txt}
~~~~
{: #fig-example1_stack2 title="Example MPLS stack after SMEP is applied."}

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} apply to this document.


# IANA Considerations

This document requests that IANA allocates a new codepoint with the name "Stateless MNA-based Egress Protection" in the "Network Action Opcodes Registry".

| MNA Opcode |  Description                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|    TBA1    |  Stateless MNA-based Egress Protection |  This document
{: #table_iana title="SMEP Opcode IANA allocation."}


--- back
