---
title: 'RATS Endorsements'
abbrev: RATS Endorsements
docname: draft-dthaler-rats-endorsements-latest
wg: RATS Working Group
stand_alone: true
ipr: trust200902
area: Security
kw: Internet-Draft
cat: info
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: D. Thaler
  name: Dave Thaler
  org: Microsoft
  email: dthaler@microsoft.com
  street: ""
  code: ""
  city: ""
  region: ""
  country: USA

--- abstract


In the IETF Remote Attestation Procedures (RATS) architecture, a Verifier
accepts Evidence and, using Appraisal Policy typically with additional
input from Endorsements and Reference Values, generates Attestation Results
in a format needed by a Relying Party.  This document explains the purpose and
role of Endorsements and discusses some considerations in the choice of
message format for Endorsements.

--- middle

# Introduction

Section 3 in the RATS Architecture {{!RFC9334}} gives an overview of the roles
and conceptual messages in the IETF Remote Attestation Architecture.
As discussed in that document, a Verifier accepts Evidence and Endorsements,
and appraises them using Appraisal Policy for Evidence, typically against
a set of Reference Values.

Various formats exist, including standard and vendor-specific formats, for
the conceptual messages shown.  Indeed, one of the purposes of a Verifer as depicted
in Figure 9 of {{RFC9334}} is to be able to accept Evidence in a variety of 
formats and generate Attestation Results in the format needed by a Relying Party.

# Current State vs Reference States {#statetypes}

Appraisal policies (Appraisal Policy for Evidence, and Appraisal Policy for
Attestation Results) involve comparing the current state of an attester against
desired or undesired states, in order to determine how trustworthy the attester
is for its purposes.  Thus, a Verifier needs to receive messages with information
about current state, and information about desired/undesired states, and an appraisal
policy that controls how the two are compared.

Current state is a group of claims about the actual state of the attester at a
given point in time.  Generally speaking, each claim has a name (or other ID)
and a singleton value, being the value of that specific attester at a given point
in time. Some claims may inherently have multiple values, such as a list of
files in a given location on the device, but for our purposes we will treat such
a list as a single unit, meaning one attester at one point in time.

Each attester in general has multiple components (e.g., hardware, firmware,
Operating System, etc.), each with their own set of claims (sometimes called
a "claimset"), where the current state of the attester is a group of such claimsets,
for all the key components of the attester that are essential to determining
trustworthiness.

Reference state is a group of claims about the desired or undesired state of
the attester.  Typically, each claim has a name (or other ID) and
a set of potential values, being the current values that are allowed/disallowed
when determining whether to trust the attester.  In general there may be more
gradation than simply "allowed or disallowed" so each value might include some
more complex level of gradation in some implementations.

That is, where current state has a single value per claim per component
applying to one device at one point in time, reference state has a set of values
per claim per component.  The appraisal policy then specifies how to match
the current value against the set of reference values.

Some examples of such matching include:
* The current value must be in the set of allowed reference values.
* The current value must not be in the set of disallowed reference values.
* The current value must be in a range where two reference values are the min and max.

## RATS Conceptual Messages

RATS conceptual messages in {{RFC9334}} fall into the above categories as follows:

* Current state: Evidence, Endorsements, Attestation Results
* Reference state: Reference Values
* Appraisal policy: Appraisal Policy for Evidence, Appraisal Policy for Attestation Results

The figure below shows an example of verifier input for a layered attester
as discussed in {{RFC9334}}.

~~~~ aasvg
             / .-------------.   Appraisal    .-----------------.  \
            |  |Current state|    Policy      | Reference state |  | R
            |  |  (layer N)  |                |    (layer N)    |  | e
            |  '-------------'       |        '-----------------'  | f
            |  .-------------.       |        .-----------------.  | e
   Evidence |  |Current state|       |        | Reference state |  | r
            |  |  (layer 2)  |       |        |    (layer 2)    |  | e 
            |  '-------------'       v        '-----------------'  | n
            |  .-------------.  <==========>  .-----------------.  | c
            |  |Current state|   Comparison   | Reference state |  | e
            |  |  (layer 1)  |     Rules      |    (layer 1)    |  |  
            \  '-------------'                '-----------------'  | V
                                                                   | a
            /  .-------------.                .-----------------.  | l
Endorsement |  |Current state|                | Reference state |  | u
            |  |  (layer 0)  |                |    (layer 0)    |  | e
            \  '-------------'                '-----------------'  / s
~~~~
{: #input artwork-align="center" title="Example Verifier Input"}

While the above example only shows one layer within Endorsements as
the typical case, there could be multiple layers within it, such as
a chip added to a hardware board potentially from a different vendor.

A Trust Anchor Store is a special case of
state above, where the Reference State would be the set of trust anchors
accepted (or rejected) by the Verifier, and the Current State would be
a trust anchor used to sign Evidence or Endorsements.

In a DICE-based layered attestation for example, the current state of each layer
is signed by a key held by the next lower layer.  Thus in the example diagram
above, the layer 2 current state (e.g., OS state) is signed by a layer 1 key
(e.g., a signing key used by the firmware), the layer 1 current state (e.g.,
firmware state) is signed by a layer 0 key (e.g., a hardware key stored in ROM),
and the layer 0 current state (hardware specs and key ID) is signed by a layer 0
key (e.g., a vendor key) which is matched against the Verifier's trust anchor
store, which is part of the layer 0 reference state depicted above.

# Endorsement Format Considerations

This section discusses considerations around formats for Endorsements.

## Security Considerations

In many scenarios, a Verifiers can also support a variety of different formats,
and while code size may not be a huge concern, simplicity and correctness of code
is essential to security.  "Complexity is the enemy of security" is a popular
security mantra and hence to increase security, any decrease in complexity
helps.  As such, using the same format for both Evidence and Endorsements
can reduce complexity and hence increase security.

## Scalability Considerations {#scalability}

We currently assume that Reference Value Providers and Endorsers typically
provide the same information to a potentially large number of clients
(Verifiers, or potentially to other entities for later relay to a Verifier),
and are generally on devices that are not constrained nodes, and hence additional
scalability, including code size, is not a significant concern.

The scenario where scalability in terms of code size is strongest, however, is
when a Verifier is embedded into a constrained node.  For example, when a constrained
node is a Relying Party for most purposes, but still needs a way to establish
trust in the Verifier it will use.  In such a case, the Relying Party may have
a constrained Verifier embedded in it that is only capable of appraising Evidence
provided by its desired Verifier.  Thus, the Relying Party uses its embedded Verifier
for purposes of appraising its desired Verifier which it treats as only an Attester,
and once verified, then uses it for verification of all other attesters.
In this scenario, the embedded Verifier may have code and data size constraints,
and a very simple (by comparison) appraisal policy and desired state (e.g.,
a required trust anchor that Evidence must be signed with and little else).

Using the same message format for Evidence, Endorsements, and (later) Attestation Results received
from the later Verifier, can provide a code size savings due to having only a single parser
in this limited case.

# IANA Considerations

This document does not require any actions by IANA.

--- back
