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
The MPLS Egress Protection Framework specifies a fast reroute framework for protecting IP/MPLS services.
To that end, bypass tunnels have to be signaled to the Point of Local Repair (PLR).
Further, the PLR must maintain the bypass forwarding state on a per-transport-tunnel basis.
This document presents the concept of Stateless MNA-based Egress Protection (SMEP).
SMEP protects egress routers by providing alternative MPLS egress labels in-stack.
This mechanism does not require a bypass forwarding state in PLRs.
An example for the application of SMEP is given using a MPLS network action for stack management.

--- middle

# Introduction
The MPLS egress protection framework in {{?RFC8679}} establishes bypass tunnels for egress routers on an egress failure, i.e., on a node or a link failure.
This is referred to as egress protection.
The protection mechanism relies on a Point of Local Repair (PLR) to perform local failure detection and local repair.
Typically, this PLR is the penultimate router.
When an egress failure occurs, packets are rerouted to an alternative egress router.
The PLR node maintains the bypass forwarding state, which is a mapping of specific labels to bypass tunnels.
The bypass tunnels are signaled using existing mechanisms, i.e., via an IGP, or topology-driven label distribution protocols such as LDP.

This document defines the concept of Stateless MNA-based Egress Protection (SMEP).
SMEP provides an alternative to the rerouting mechanism defined for the PLR, allowing the PLR to be stateless.

## Terminology

{::boilerplate bcp14-tagged}

### Abbreviations
This document makes use of the terms defined in {{?RFC8679}} and in {{?I-D.ietf-mpls-mna-hdr}}.

Further abbreviations used in this document:

| Abbreviation | Meaning                               | Reference     |
| ------------ | ------------------------------------- | ------------- |
| BML          | Bypass MPLS Label                     | This document |
| SMEP         | Stateless MNA-based Egress Protection | This document |
{: #table_abbrev title="Abbreviations."}

# Concept of SMEP
With SMEP, the egress bypass tunnel is indicated by one or multiple alternative MPLS forwarding labels which are located at the bottom of stack.
We call those labels Bypass MPLS Labels (BMLs).
On an egress failure, SMEP uses the BMLs in the stack to protect the egress tunnel.
The PLR node is required to install the MPLS forwarding entries for the bypass tunnels using the BMLs.
However, it does not need to maintain a table that maps transport tunnels to backup paths.
Likewise, the PLR is not involved in the signaling of such information.
Instead, this information is supplied in the MPLS stack from the ingress node to the PLR.
Signaling is only needed between ingress, egress, and the protector, but not with the PLR anymore.
Details of the signaling are not contained in this draft.
The general concepts and mechanisms described in {{?RFC8679}} still apply.

# Stack Management with POP-N-LSE Operation Network Action
The MPLS Network Action (MNA) framework encodes network actions and their data for processing by MPLS nodes.
{{?I-D.ietf-mpls-mna-hdr}} defines the encoding of such network actions and their data in so-called Network Action Substacks (NAS) in the MPLS stack.

{{?I-D.ihle-mpls-mna-stack-management-00}} introduces a network action for stack management.
It features a POP-N-LSE operation that pops a number `POP-N` of LSEs below the NAS.
The POP-N-LSE operation facilitates the application of SMEP.
To that end, the POP-N-LSE operation is added directly above the BMLs where `POP-N` is the number of labels in the bypass tunnel.
The processing at the PLR is as follows:

1. If the PLR does not detect an egress failure
   - The PLR executes the POP-N-LSE action and pops all BMLs.
   - The packet is forwarded as usual to the egress node.
2. If the PLR detects an egress failure
   - The POP-N-LSE action is ignored and is popped along with the top-of-stack label.
   - The BML is now the top-of-stack label. The packet is forwarded to the protector based on the BML.

# Example
A simple example topology using MNA-based egress protection with an SR bypass tunnel is shown in {{fig-smep}}.
The example network contains an LSP R1-R2-R3 and a bypass tunnel R2-R3'-R3''.

The MPLS stack using SMEP pushed by the ingress LER for the example topology is shown in {{fig-smep-example1_stack}}.
The label stack contains the forwarding labels L1, L2, L3, L3', and L3'', and the POP-N-LSE operation destined for the PLR.
Since there are two BMLs to reach the protector, `POP-N = 2`.
Labels L1 and L2 are used to forward to the penultimate router.
Label L3 is used to route to the egress node, and labels L3' and L3'' are used to route to the protector.
If the egress link or router R3 fails, the PLR can use the bypass tunnel of router R3' and R3''.

~~~~
{::include ./drawings/smep-example1.txt}
~~~~
{: #fig-smep title="Example using the POP-N-LSE operation for SMEP."}

~~~~
{::include ./drawings/smep-example1_stack.txt}
~~~~
{: #fig-smep-example1_stack title="MPLS stack at R1."}

If there is no egress failure, the LSR R2 executes the POP-N-LSE action with `POP-N = 2` and pops the BMLs L3' and L3''.
R2 pops the top-of-stack label L3 and forwards the packet as usual to the egress.

If the LSR R2 detects an egress failure, it becomes the PLR.
The POP-N-LSE action is ignored and the NAS is popped along with the top-of-stack label.
This stack is shown in {{fig-smep}}.
This time, the BMLs L3' and L3'' are at the top-of-stack.
The packet is forwarded according to those labels to the alternative egress node R3''.

~~~~
{::include ./drawings/smep-example1_stack2.txt}
~~~~
{: #fig-example1_stack2 title="MPLS stack after R3 on egress failure."}

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} and in {{?RFC8679}} apply to this document.


# IANA Considerations

This document makes no request of IANA.

--- back
