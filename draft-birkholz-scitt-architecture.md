---
title: An Architecture for Trustworthy and Transparent Digital Supply Chains
abbrev: SCITT Architecture
docname: draft-birkholz-scitt-architecture-latest
stand_alone: true
ipr: trust200902
area: Security
wg: TBD
kw: Internet-Draft
cat: std
consensus: yes
submissiontype: IETF
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes
kramdown_options:
  auto_id_prefix: sec-

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: A. Delignat-Lavaud
  name: Antoine Delignat-Lavaud
  organization: Microsoft Research
  street: 21 Station Road
  code: 'CB1 2FB'
  city: Cambridge
  email: antdl@microsoft.com
  country: UK
- ins: C. Fournet
  name: Cedric Fournet
  organization: Microsoft Research
  street: 21 Station Road
  code: 'CB1 2FB'
  city: Cambridge
  email: fournet@microsoft.com
  country: UK

normative:
  RFC8610: CDDL
  RFC8152: COSE
  RFC9162: CT
  RFC6838:
informative:
  I-D.birkholz-scitt-receipts: RECEIPTS
  PBFT:
    title: Practical byzantine fault tolerance and proactive recovery
    target: https://doi:10.1145/571637.571640
    author:
    -
      ins: M. Castro
      name: Miguel Castro
      org: Microsoft Research
    -
      ins: B. Liskov
      name: Barbara Liskov
      org: MIT Laboratory for Computer Science
    date: 2002-11
    seriesinfo:
      ACM Transactions on Computer Systems, Volume 20, Issue 4

venue:
  type: non-WG
  mail: scitt@ietf.org

--- abstract

Traceability of physical and digital artifacts in supply chains is a long-standing, but increasingly serious security concern. The rise in popularity of verifiable data structures as a mechanism to make actors more accountable for breaching their compliance promises has found some successful applications to specific use cases (such as the supply chain for digital certificates), but lacks a generic and scalable architecture that can address a wider range of use cases.

This memo defines a generic and scalable architecture to enable transparency across any supply chain with minimum adoption barriers for producers (who can register their claims on any TS, with the guarantee that all consumers will be able to verify them) and enough flexibility to allow different implementations of Transparency Services with various auditing and compliance requirements.

--- middle

# Introduction

This document describes a scalable and flexible decentralized architecture to enhance auditability and accountability in various existing and emerging supply chains.
It achieves this goal by enforcing the following complementary security guarantees:

1. statements made by issuers about supply chain artifacts must be identifiable, authentic, and non-repudiable;
2. such statements must be registered on a secure append-only ledger so that their provenance and history can be independently and consistently audited;
3. issuers can efficiently prove to any other party the registration of their claims; verifying this proof ensures that the issuer is consistent and non-equivocal when making claims.

The first guarantee is achieved by requiring issuers to sign their statements and associated metadata using a distributed public key infrastructure. The second guarantee is achieved by storing the signed statement on an immutable, append-only, transparent ledger. The last guarantee is achieved by implementing the ledger using a verifiable data structure (such as a Merkle Tree), and by requiring a transparency service (TS) that operates the ledger to endorse its state at the time of registration.

The guarantees and techniques used in this document generalize those of Certificate Transparency {{-CT}}, which can be re-interpreted as an instance of this architecture for the supply chain of X.509 certificates. However, the range of use cases and applications in this document is much broader, which requires much more flexibility in how each TS implements and operates its ledger. Each service may enforce its own policy for authorizing entities to register their claims on the TS. Some TS may also enforce access control policies to limit who can audit the full ledger, or keep some information on the ledger encrypted. Nevertheless, it is critical to provide global interoperability for all TS instances as the composition and configuration of involved supply chain entities and their system components is ever changing and always in flux.

A TS provides visibility into claims issued by supply chain entities and their sub-systems. These claims are called Digital Supply Chain Artifacts (DSCA).
A TS vouches for specific and well-defined metadata about these DSCAs. Some metadata is selected (and signed) by the issuer, indicating, e.g., "who issued the DSCA" or "what type of DSCA is described" or "what is the DSCA version"; whereas additional metadata is selected (and countersigned) by the TS, indicating, e.g., "when was the DSCA registered in the ledger". The DSCA contents can be opaque to the TS, if so desired: it is the metadata that must always be transparent in order to warrant trust.

Transparent claims provide a common basis for holding issuers accountable for the DSCA they release and (more generally) principals accountable for auxiliary claims they make about DSCAs. Hence, issuers may register new claims about their artifacts, but they cannot delete or alter earlier claims, or hide their claims from third parties such as auditors.

Trust in the TS itself is supported both by protecting their implementation (using, for instance, replication, trusted hardware, and remote attestation of systems) and by enabling independent audits of the correctness and consistency of its ledger, thereby holding the organization accountable that operates it. Unlike CT, where independent auditors are responsible for enforcing the consistency of multiple independent instances of the same global ledger, we require each TS to guarantee the consistency of its own ledger (for instance, through the use of a consensus algorithm between replicas of the ledger), but assume no consistency between different transparency services.

The TS specified in this architecture caters to two types of audiences:

1. DSCA Issuers: entities, stakeholders, and users involved in supply chain interactions that need to release DSCAs to a definable set of peers; and
2. DSCA Consumers: entities, stakeholders, and users involved in supply chain interactions that need to access, validate, and trust DSCAs.

DSCA Issuers rely on being discoverable and represented as the responsible parties for released DSCAs by the TS in a believable manner.
Analogously, DSCA Consumers rely on verifiable trustworthiness assertions associated with DSCAs and their processing in a believable manner.
If trust can be put into the operations that record DSCAs in a secure, append-only ledger via an online operation, the same trust can be put into a corresponding receipt that is the result of these online operations issued by the TS and that can be validated in offline operations.

The TS specified in this architecture can be implemented by various different types of services in various types of languages provided via various variants of API layouts.

The global interoperability enabled and guaranteed by the TS is enabled via core components (architectural constituents) that come with prescriptive requirements (that are typically hidden away from the user audience via APIs). The core components are based on the Concise Signing and Encryption standard specified in {{-COSE}}, which is used to sign released DSCAs and to build and maintain a Merkle tree that functions as the append-only ledger for DSCAs.
The format and verification process for ledger-based transparency receipts are described in {{-RECEIPTS}}.

## Requirements Notation

{::boilerplate bcp14-tagged}

# Use Cases

This section presents representative and solution-agnostic use cases to illustrate the scope of SCITT and the processing of Digital Supply Chain Artifacts.

## Software Bill of Materials (SBOM)

As the ever increasing complexity of large software projects requires more modularity and abstractions to manage them, keeping track of their full Trusted Computing Base (TCB) is becoming increasingly difficult. Each component may have its own set of dependencies and libraries. Some of these dependencies are binaries, which means their TCB depends not only on their source, but also on their build environment (compilers and tool-chains). Besides, many source and binary packages are distributed through various channels and repositories that may not be trustworthy.

Software Bills of Materials (SBOM) help the authors, packagers, distributors, auditors and users of software understand its provenance and who may have the ability to introduce a vulnerability that can affect the supply chain downstream. However, the usefulness of SBOM in protecting end users is limited if supply chain actors cannot be held accountable for their contents. For instance, consider a package repository for an open source operating system distribution. The operator of this repository may decide to provide a malicious version of a package only to users who live in a specific country. They can write two equivocal SBOMs for the honest and backdoored versions of the package, so that nobody outside the affected country can discover the malicious version, but victims are not aware they are being targeted.

## Confidential Computing

Confidential Computing can leverage hardware-protected trusted execution environments (TEEs) to operate cloud services that protect the confidentiality of data that they process. It relies on remote attestation, which allows the service to prove to remote users what is the hash of its software, as measured and signed by the hardware.

For instance, consider a speech recognition service that implements machine learning inference using a deep neural network model. The operator of the service wants to prove to its users that the service preserves the user's privacy, that is, the submitted recordings can only be used to detect voice commands but no other purpose (such as storing the recordings or detecting mentions of brand names for advertisement purposes).
When the user connects to the TEE implementing the service, the TEE presents attestation evidence that includes a hardware certificate and a software measurement for their task; the user verifies this evidence before sending its recording.

But how can users verify the software measurement for their task? And how can operators update their service, e.g., to mitigate security vulnerabilities or improve accuracy,
without first convincing all users to update the measurements they trust?

A supply chain that maintains a transparent record of the successive software releases for machine-learning models and runtimes, recording both their software measurements
and their provenance (source code, build reports, audit reports,...) can provide users with the information they need to authorize these tasks, while holding the service operator accountable for the software they release for them.

## Cold Chains for Seafood

Once seafood is caught, its quality is determined -- amongst other criteria -- via the integrity of a cold chain that ensures a regulatory perspective freshness mandating a continuous storing temperature between 1 {{{°}}}C and 0 {{{°}}}C (or -18 {{{°}}}C and lower for frozen seafood). The temperature is recorded by cooling units adhering to certain compliance standards automatically. Batches of seafood can be split or aggregated before arriving in a shelf so that each unit can potentially have a potentially unique cold chain record whose transparency impacts the accuracy of the shelf-life associated with it. Especially in early links of the supply chain, Internet connection or sophisticated IT equipment are typically not available and sometimes temperature measurements are recorded manually and digital records are created in hindsight.

# Terminology

The terms defined in this section have special meaning in the context of Supply Chain Integrity, Transparency, and Trust throughout this document. When used in text, the corresponding terms are capitalized. To ensure readability, only a core set of terms is included in this section.

Artifact:

: the physical or non-physical item that is moving along the supply chain.

Statement:

: any serializable information about an Artifact. To help interpretation of Statements, they must be tagged with a media type (as specified in {{RFC6838}}).

Claim:

: an identifiable and non-repudiable Statement about an Artifact made by an Issuer. In SCITT, Claims are encoded as COSE signed objects; the payload of the COSE structure contains the Statement.

Issuer:

: creator of Claims submitted to a Transparency Service for Registration. The Issuer may be the owner or author of the Artifact, or a completely independent third party.

Envelope:

: the metadata added to the Statement by the Issuer to make it a Claim. It contains the identity of the Issuer and other information to help Verifiers identify the Artifact referred in the Statement. A Claim binds the Envelope to the Statement. In COSE, the Envelope consists of protected headers.

Feed:

: An identifier chosen by the Issuer for the Artifact. For every Issuer and Feed, the Ledger on a Transparency Service contains a sequence of Claims about the same Artifact.
 In COSE, Feed is one header attributes in the protected header of the Envelope.

Ledger:

: the verifiable data structure that stores Claims in a transparency service. SCITT supports multiple Ledger formats to accommodate different transparency service implementations, such as historical Merkle Trees and sparse Merkle Trees.

Transparency Service:

: the entity that maintains and extends the Ledger, and endorses its state. A Transparency Service can be a complex distributed system, and SCITT requires the TS to provide many security guarantees about its Ledger. The identity of a TS is captured by a public key that must be known by Verifiers in order to validate Receipts.

Receipt:

: a Receipt is a special form of COSE countersignature for Claims that embeds cryptographic evidence that the Claim is recorded in the Ledger. It consists of a Ledger-specific inclusion proof, a signature by the Transparency Service of the state of the Ledger, and additional metadata (contained in the countersignature protected headers) to assist in auditing.

Registration:

: the process of submitting a Claim to a Transparency Service, applying its registration policy, storing it in the Ledger and producing the Receipt returned to the submitter.

Transparent Claim:

: a Claim that is augmented with a Receipt of its registration. A Transparent Claim remains a valid Claim (as the Receipt is carried in the countersignature), and may be registered again in a different TS.

Verifier:

: the entity that consumes Transparent Claims, verifying their proofs and inspecting their Statements, either before using their Artifacts, or later to audit their supply chain.

{: #mybody}

# Definition of Transparency

In this document, we use a definition of transparency built over abstract notions of Ledgers and Receipts. Existing transparency systems such as Certificate Transparency are instances of this definition.

A Claim is an identifiable and non-repudiable Statement made by an Issuer. The Issuer selects additional metadata and attaches a proof of endorsement (in most cases, a signature) using the identity key of the Issuer that binds the Statement and its metadata. Claims can be made transparent by attaching a proof of Registration by a TS, in the form of a Receipt that countersigns the Claim and witnesses its inclusion in the Ledger of a TS. By extension, we may say an Artifact (e.g. a firmware binary) is transparent if it comes with one or more Transparent Claims from its author or owner, though the context should make it clear what type of Claim is expected for a given Artifact.

Transparency does not prevent dishonest or compromised Issuers, but it holds them accountable: any Artifact that may be used to target a particular user that checks for Receipts must have been recorded in the tamper-proof Ledger, and will be subject to scrutiny and auditing by other parties.

Transparency is implemented by a Ledger that provides a consistent, append-only, publicly available record of entries. Implementations of TS may protect their Ledger using a combination of trusted hardware, replication and consensus protocols, and cryptographic evidence. A Receipt is an offline, universally-verifiable proof that an entry is recorded in the edger. Receipts do not expire, but it is possible to append new entries that subsume older entries.

Anyone with access to the Ledger can independently verify its consistency and review the complete list of Claims registered by each Issuer. However, the Ledgers of separate Transparency Services are generally disjoint, though it is possible to take a Claim from one Ledger and register it again on another (if its policy allows it), so the authorization of the Issuer and of the Ledger by the Verifier of the Receipt are generally independent.

Reputable Issuers are thus incentivized to carefully review their Statements before signing them into Claims. Similarly, reputable TS are incentivized to secure their Ledger, as any inconsistency can easily be pinpointed by any auditor with read access to the Ledger. Some Ledger formats may also support consistency auditing through Receipts, that is, given two valid Receipts the TS may be asked to produce a cryptographic proof that they are consistent. Failure to produce this proof can indicate that the TS operator misbehaved.

# Architecture Overview

~~~~ aasvg
               Artifact
                  |
                  v                       .------------------.
Issuer       --> Statement    Envelope    | DID Key Manifest |
                   \           /          | (decentralized)  |
                    \         /           '--+---------+-----'
                     \_______/               |         |
                         |                   |         |
                         v        signature  |         |
                       Claim  <-------------'          |
                         |                             |
                         |   Receipt   .--------.      |
Transparency -->         +-------------+ Ledger +-.    |
Service                  |             '--------'  |   |
                         v                         |   |
                    Transparent                    |   |
                       Claim --.          .-----------'
                         |      |        |         |
                         |      |        |         |
                         |      |        |         |
Verifier    -->          |     Verify Claim        |
                         |                         |
Auditor     -->        Collect Receipts     Replay Ledger
~~~~

The SCITT architecture consists of a very loose federation of Transparency Services, and a set of common formats and protocols for issuing, registering and auditing Claims.
In order to accomodate as many TS implementations as possible, this document only specifies the format of Claims (which must be used by all Issuers) and a very thin wrapper format for Receipts, which specifies the TS identity and the Ledger algorithm. Most of the details of the Receipt's contents are specific to the Ledger algorithm. The {{-RECEIPTS}} document defines two initial Ledger algorithms (for historical and sparse Merkle Trees), but other Ledger formats (such as blockchains, or hybrid historical and indexed Merkle Trees) may be proposed later.

In this section, we describe at a high level the three main roles and associated processes in SCITT: Issuers and the Claim issuance process, transparency Ledgers and the Claim Registration process, and Verifiers and the Receipt validation process.

## Claim Issuance

### Issuer Identity

Before an Issuer is able to produce Claims, it must first create its [decentralized identifier](https://www.w3.org/TR/did-core) (also known as a DID).
A DID can be *resolved* into a *key manifest* (a list of public keys indexed by a *key identifier*) using many different DID methods.

Issuers MAY choose the DID method they prefer, but with no guarantee that all TS will be able to register their Claim. To facilitate interoperability, all Transparency Service implementations SHOULD support the `did:web` method from [https://w3c-ccg.github.io/did-method-web/]. For instance, if the Issuer publishes its manifest at `https://sample.issuer/user/alice/did.json`, the DID of the Issuer is `did:web:sample.issuer:user:alice`.

Issuers SHOULD use consistent decentralized identifiers for all their Artifacts, to simplify authorization by Verifiers and auditing. They MAY update their DID manifest, for instance to refresh their signing keys or algorithms, but they SHOULD NOT remove or change any prior keys unless they intend to revoke all Claims issued with those keys. This DID appears in the Issuer header of the Claim's Envelope, while the version of the key from the manifest used to sign the Claim is written in the `kid` header.

### Naming Artifacts

Many Issuers issue Claims about different Artifacts under the same DID, so it is important for everyone to be able to immediately recognize by looking at the Envelope of a Claim what Artifact it is referring to. This information is stored in the Feed header of the Envelope. Issuers MAY use different signing keys (identified by `kid` in the resolved key manifest) for different Artifacts, or sign all Claims under the same key.

### Claim Metadata

Besides Issuer, Feed and kid, the only other mandatory metadata in the Claim is the type of the Payload, indicated in the `cty` Envelope header.
However, this set of mandatory metadata is not sufficient to express many important Registration policies. For example, a Ledger may only allow a Claim to be registered if it was signed recently. While the Issuer is free to add any information in the payload of the Claim, the TS (and most of its auditor) can only be expected to interpret information in the Envelope.

Such metadata, meant to be interpreted by the TS during Registration policy evaluation, should be added to the `reg_info` header. While the header MUST be present in all Claims, its contents consist of a map of named attributes. Some attributes (such as the Issuer's timestamp) are standardized with a defined type, to help uniformize their semantics across TS. Others are completely customizable and may have arbitrary types. In any case, all attributes are optional so the map MAY be empty.

## Transparency Service (TS)

The role of TS can be decomposed into several major functions. The most important is maintaining a Ledger, the verifiable data structure that records Claims, and enforcing a Registration policy. It also maintains a service key, which is used to endorse the state of the Ledger in Receipts. All TS MUST expose standard endpoints for Registration of Claims and Receipt issuance, which is described in {{sec-messages}}. Each TS also defines its Registration policy, which MUST apply to all entries in the Ledger.

The combination of Ledger, identity, Registration policy evaluation, and Registration endpoint constitute the trusted part of the TS. Each of these components SHOULD be carefully protected against both external attacks and internal misbehavior by some or all of the operators of the TS. For instance, the code for policy evaluation, Ledger extension and endorsement may be protected by running in a TEE; the Ledger may be replicated and a consensus algorithm such as Practical Byzantine Fault Tolerance (pBFT {{PBFT}}) may be used to protect against malicious or vulnerable replicas; threshold signatures may be use to protect the service key, etc.

Beyond the trusted components, Transparency Services may operate additional endpoints for auditing, for instance to query for the history of Claims made by a given Issuer and Feed. Implementations of TS SHOULD avoid using the service identity and extending the Ledger in auditing endpoints; as much as practical, the Ledger SHOULD contain enough evidence to re-construct verifiable proofs that the results returned by the auditing endpoint are consistent with a given state of the Ledger.

### Service Identity, Remote Attestation, and Keying

Every TS MUST have a public service identity,
associated with public/private key pairs for signing on behalf of the service. In particular, this identity must be known by Verifiers when validating a Receipt

This identity should be stable for the lifetime of the service, so that all Receipts remain valid and consistent. The TS operator MAY use a distributed identifier as their public service identity if they wish to rotate their keys, if the Ledger algorithm they use for their Receipt supports it. Other types of cryptographic identities, such as parameters for non-interactive zero-knowledge proof systems, may also be used in the future.

The TS SHOULD provide evidence that it is securely implemented and operated, enabling remote authentication of the hardware platforms and/or software TCB that run the TS. This additional evidence SHOULD be recorded in the Ledger and presented on demand to Verifiers and auditors.

For example, consider a TS implemented using a set of replicas, each running within its own hardware-protected trusted execution environments (TEEs). Each replica SHOULD provide a recent attestation report for its TEE, binding their hardware platform to the software that runs the Transparency Service, the long-term public key of the service, and the key used by the replica for signing Receipts. This attestation evidence SHOULD be supplemented with transparency Receipts for the software and configuration of the service, as measured in its attestation report.

### Registration Policies

> **Editor's note**
>
> The initial version of this document assumes Registration policies are set for the lifetime of the Ledger, and that they apply to all Issuers and Feeds uniformly.
> There is an ongoing discussion on how to make the design more flexible to allow per-Issuer and per-Feed Registration policies, and whether such policies should be updatable or if a policy change requires a Feed change.
> Please contribute your comments to the SCITT mailing list.

Each TS is initially configured with a set of Registration policies, which will be applied for the lifetime of the Ledger.
A Registration policy represents a predicate that takes as input the current Ledger and the Envelope of a new Claim to register (including the `reg_info` header which contains customizable additional attributes), and returns a Boolean decision on whether the Claim should be included on the Ledger or not. A TS MUST ensure that all its Registration policies return a positive decision before adding a Claim to the Ledger.

While Registration policies are a burden for Issuers (some may require them to maintain state to remember what they have signed before) they support stronger transparency guarantees, and they greatly help Verifiers and auditors in making sense of the information on the Ledger. (This is particularly relevant for parties that verify Receipts on their own, without accessing the Ledger.) For instance, if a TS doesn't apply any policy, Claims may be registered in a different order than they have been issued, and old Claims may be replayed, which makes it difficult to understand the logical history of an Artifact, or to prevent rollback attacks.

There are two kinds of Registration policies: (1) named policies have standardized semantics that are uniform across all implementations of SCITT Transparency Services, while (2) custom policies are opaque and may contain pointers to (or even inlined) policy descriptions (declarative or programmable).

Transparency services MUST advertise the Registration policies enforced by their service, including the list of `reg_info` attributes they require, both to minimize the risk of rejecting Claims presented by Issuers, and to advertise the properties implied by Receipt verification. Implementations of Receipt Verifiers SHOULD persist the list of Registration policies associated with a service identity, and return the list of Registration policies as an output of Receipt validation. Auditors MUST re-apply the Registration policy of every entry in the Ledger to ensure that the Ledger applied them correctly.

Custom policies may use additional information present in the Ledger outside of Claims. For instance, Issuers may have to register on the TS before Claims can be accepted; a custom policy may be used to enforce access control to the Transparency Service. Verifying the signature of the Issuer is also a form of Registration policy, but it is globally enforced in order to separate authentication and authorization, with policy only considering authentic inputs.

{{tbl-initial-named-policies}} defines an initial set of named policies that TS may decide to enforce. This may be evolved in future drafts.

Policy Name | Required `reg_info` attributes | Implementation
---|---|---
TimeLimited | `register_by: uint` | Returns true if now () < register_by. The Ledger MUST store the Ledger time at Registration along with the Claim, and SHOULD indicate it in Receipts
Sequential | `sequence_no: uint` | First, lookup in the Ledger for Claims with the same Issuer and Feed. If at least one is found, returns true if and only if the `sequence_no` of the new Claim is the highest `sequence_no` in the existing Claims incremented by one. Otherwise, returns true if and only if `sequence_no = 0`.
Temporal | `issuance_ts: uint` | Returns true if and only if there is no Claim in the Ledger with the same Issuer and Feed with a greater `issuance_ts`
NoReplay | None | Returns true if and only if the Claim doesn't already appear in the Ledger
{: #tbl-initial-named-policies title="An Initial Set of Named Policies"}

### Ledger Security Requirements

There are many different candidate verifiable data structures that may be used to implement the Ledger, such as chronological Merkle Trees, sparse/indexed Merkle Trees, full blockchains, and many other variants. We only require the Ledger to support concise Receipts (i.e. whose size grows at most logarithmically in the number of entries in the Ledger). This does not necessarily rule out blockchains as a Ledger, but may necessitate advanced Receipt schemes that use arguments of knowledge and other verifiable computing techniques.

Since the details of how to verify a Receipt are specific to the data structure, we do not specify any particular Ledger format in this document. Instead, we propose two initial formats for Ledgers in {{-RECEIPTS}} using historical and sparse Merkle Trees. Beyond the format of Receipts, we require generic properties that should be satisfied by the components in the TS that have the ability to write to the Ledger.

#### Finality

The Ledger is append-only: once a Claim is registered, it cannot be modified, deleted, or moved. In particular, once a Receipt is returned for a given Claim, the Claim and any preceding entry in the Ledger become immutable, and the Receipt provides universally-verifiable evidence of this property.

#### Consistency

There is no fork in the Ledger: everyone with access to its contents sees the same sequence of entries, and can check its consistency with any Receipts they have collected.
TS implementations SHOULD provide a mechanism to verify that the state of the Ledger encoded in an old Receipt is consistent with the current Ledger state.

#### Replayability and Auditing

Everyone with access to the Ledger can check the correctness of its contents. In particular,

- the TS defines and enforces deterministic Registration policies that can be re-evaluated based solely on the contents of the Ledger at the time of registraton, and must then yield the same result.

- The ordering of entries, their cryptographic contents, and the Ledger governance may be non-deterministic, but they must be verifiable.

- The TS SHOULD store evidence about the resolution of distributed identifiers into manifests.

- The TS MAY additionally support verifiability of client authentication and access control.

#### Governance and Bootstrapping

The TS needs to support governance, with well-defined procedures for allocating resources to operate the Ledger (e.g., for provisioning trusted hardware and registering their attestation materials in the Ledger) and for updating its code (e.g., relying on Transparent Claims about code updates, secured on the Ledger itself, or on some auxiliary TS).

Governance procedures, their auditing, and their transparency are implementation specific. The TS SHOULD document them.

- Governance may be based on a consortium of members that are jointly responsible for the TS, or automated based on the contents of an auxiliary governance TS.

- Governance typically involves additional records in the Ledger to enable its auditing. Hence, the Ledger may contain both Transparent Claims and governance entries.

- Issuers, Verifiers, and third-party auditors may review the TS governance before trusting the service, or on a regular basis.

## Verifying Transparent Claims {#validation}

For a given Artifact, Verifiers take as trusted inputs:

1. the distributed identifier of the Issuer (or its resolved key manifest),
2. the expected name of the Artifact (i.e. the Feed),
3. the list of service identities of trusted TS.

When presented with a Transparent Claim for the Artifact, they verify its Issuer identity, signature, and Receipt.
They may additionally apply a validation policy based on the protected headers present both in the Envelope or in the countersignature and the Statement itself, which may include security-critical Artifact-specific details.

Some Verifiers may systematically resolve the Issuer DID to fetch their latest DID document. This strictly enforces the revocation of compromised keys: once the Issuer has updated its document to remove a key identifier, all Claims signed with this `kid` will be rejected. However, others may delegate DID resolution to a trusted third party and/or cache its results.

Some Verifiers may decide to skip the DID-based signature verification, relying on the TS's Registration policy and the scrutiny of other Verifiers. Although this weakens their guarantees against key revocation, or against a corrupt TS, they can still keep the Receipt and blame the Issuer or the TS at a later point.

# Claim Issuance, Registration, and Verification

This section details the interoperability requirements for implementers of Claim issuance and validation libraries, and of Transparency Services.

##  Envelope and Claim Format

The formats of Claims and Receipts are based on CBOR Object Signing and Encryption (COSE). The choice of CBOR is a trade-off between safety (in particular, non-malleability: each Claim has a unique serialization), ease of processing and availability of implementations.

At a high-level that is the context of this architecture, a Claim is a COSE single-signed object (i.e. `COSE_Sign1`) that contains the correct set of protected headers. Although Issuers and relays may attach unprotected headers to Claims, Transparency Services and Verifiers MUST NOT rely on the presence or value of additional unprotected headers in Claims during Registration and validation.

All Claims MUST include the following protected headers:

- algorithm (label: `1`): Asymmetric signature algorithm used by the Claim Issuer, as an integer, for example `-35` for ECDSA with SHA-384, see [COSE Algorithms registry](https://www.iana.org/assignments/cose/cose.xhtml);
- Issuer (label: `TBD`, to be registered): DID (Decentralized Identifier, see [W3C Candidate Recommendation](https://www.w3.org/TR/did-core/)) of the signer, as a string, for example `did:web:example.com`;
- Feed (label: `TBD`): the Issuer's name for the Artifact, as a string;
- payload type (label: `3`): Media type of payload as a string, for example `application/spdx+json`
- Registration policy info (label: `TBD`): a map of additional attributes to help enforce Registration policies;
- DID key selection hint (label: `TBD`): a DID method-specific selector for the signing key, as a bytestring.

Additionally, Claims MAY carry the following unprotected headers:

- Receipts (label: `TBD`, to be registered): Array of Receipts, defined in {{-RECEIPTS}}

In CDDL {{-CDDL}} notation, the Envelope is defined as follows:

~~~~ cddl
SCITT_Envelope = COSE_Sign1_Tagged

COSE_Sign1_Tagged = #6.18(COSE_Sign1)

COSE_Sign1 = [
  protected : bstr .cbor Protected_Header,
  unprotected : Unprotected_Header,
  payload : bstr,
  signature : bstr
]

Reg_Info = {
  ? "register_by": uint,
  ? "sequence_no": uint,
  ? "issuance_ts": uint,
  * tstr => any
}

; All protected headers are mandatory, to protect against faulty implementations of COSE
; that may accidentally read a missing protected header from the unprotected headers.
Protected_Header = {
  1 => int               ; algorithm identifier
  3 => tstr              ; payload type
  258 => tstr            ; DID of Issuer
  259 => tstr            ; Feed
  260 => Reg_Info        ; Registration policy info
  261 => bstr            ; key selector
}

Unprotected_Header = {
   ? 257 => SCITT_Receipt / [+ SCITT_Receipt]
}
~~~~

## Claim Issuance

There are many types of Statements (such as SBOMs, malware scans, audit reports, policy definitions) that Issuers may want to turn into Claims. The Issuer must first decide on a suitable format to serialize the Statement, such as:
- JSON-SPDX
- CBOR-SPDX
- SWID
- CoSWID
- CycloneDX
- in-toto
- SLSA

Once the Statement is serialized with the correct content type, the Issuer should fill in the attributes for the Registration policy information header. From the Issuer's perspective, using attributes from named policies ensures that the Claim may only be registered on Transparency Services that implement the associated policy. For instance, if a Claim is frequently updated, and it is important for Verifiers to always consider the latest version, Issuers SHOULD use the `sequence_no` or `issuer_ts` attributes.

Once all the Envelope headers are set, the Issuer MAY use a standard COSE implementation to produce the serialized Claim (the SCITT tag of `COSE_Sign1_Tagged` is outside the scope of COSE, and used to indicate that a signed object is a Claim).

## Registering Signed Claims

The same Claim may be independently registered in multiple TS. To register a Claim, the service performs the following steps:

1. Client authentication. This is implementation-specific, and MAY be unrelated to the Issuer identity. Claims may be registered by a different party than their Issuer.

2. Issuer identification. The TS MUST store evidence of the DID resolution for the Issuer protected header of the Envelope and the resolved key manifest at the time of Registration for auditing. This MAY require that the service resolve the Issuer DID and record the resulting document, or rely on a cache of recent resolutions.

3. Envelope signature verification, as described in COSE signature, using the signature algorithm and verification key of the Issuer DID document.

4. Envelope validation. The service MUST check that the Envelope has a payload and the protected headers listed above. The service MAY additionally verify the payload format and content.

5. Apply Registration policy: for named policies, the TS should check that the required Registration info attributes are present in the Envelope and apply the check described in Table 1. A TS MUST reject Claims that contain an attribute used for a named policy that is not enforced by the service. Custom Claims are evaluated given the current Ledger state and the entire Envelope, and MAY use information contained in the attributes of named policies.

6. Commit the new Claim to the Ledger

7. Sign and return the Receipt.

The last two steps MAY be shared between a batch of Claims recorded in the Ledger.

The service MUST ensure that the Claim is committed before releasing its Receipt, so that it can always back up the Receipt by releasing the corresponding entry in the Ledger. Conversely, the service MAY re-issue Receipts for the Ledger content, for instance after a transient fault during Claim Registration.

## Validation of Transparent Claims

This section provides additional implementation considerations, the high-level validation algorithm is described in {{validation}}, with the Ledger-specific details of checking Receipts are covered in {{-RECEIPTS}}.

Before checking a Claim, the Verifier must be configured with one or more identities of trusted Transparency Services. If more than one service is configured, the Verifier MUST return which service the Claim is registered on.

In some scenarios, the Verifier already expects a specific Issuer and Feed for the Claim, while in other cases they are not known in advance and can be an output of validation. Verifiers SHOULD offer a configuration to decide if the Issuer's signature should be locally verified (which may require a DID resolution, and may fail if the manifest is not available or if the key is revoked), or if it should trust the validation done by the TS during Registration.

Some Verifiers MAY decide to locally re-apply some or all of the Registration policies if they have limited trust in the TS. In addition, Verifiers MAY apply arbitrary validation policies after the signature and Receipt have been checked. Such policies may use as input all information in the Envelope, the Receipt, and the payload, as well as any local state.

Verifiers SHOULD offer options to store or share Receipts in case they are needed to audit the TS in case of a dispute.

# Federation

We explain how multiple, independent Transparency Services can be composed to distribute supply chains without a single transparency authority trusted by all parties.

Multiple SCITT instances, governed and operated by different organizations.

For example,
- a small, simple SCITT instance may keep track specifically of the software used for operating SCITT services.
- an air-gapped data center may operate its own SCITT Ledger to retain full control and auditing of its software supplies.

How?
- Policy-based. Within an organization, local Verifiers contact an authoritative SCITT that records the latest policies associated with classes of Artifacts; these policies indicate which Issuers and Ledgers are trusted for verifying signed Transparent Claims for these Artifacts.

- Other federation mechanisms?

We'd like to attach multiple Receipts to the same signed Claims, each Receipt endorsing the Issuer signature and a subset of prior Receipts. This involves down-stream Ledgers verifying and recording these Receipts before issuing their own Receipts.

# Transparency Service API

Editor's Note: this may be moved to appendix.

## Messages

### Register Signed Claims

#### Request

~~~
POST <Base URL>/entries
~~~

Body: SCITT COSE_Sign1 message

#### Response

One of the following:

- HTTP Status `201` - Registration was tentatively successful pending service consensus.
- HTTP Status `400` - Registration was unsuccessful.
  - Error code `AwaitingDIDResolutionTryLater`
  - Error code `InvalidInput`

[TODO] Use 5xx for AwaitingDIDResolutionTryLater

The `201` response contains the `x-ms-ccf-transaction-id` HTTP header which can be used to retrieve the Registration Receipt with the given transaction ID. [TODO] this has to be made generic

[TODO] probably a bad idea to define a new header, or is it ok? can we register a new one? https://www.iana.org/assignments/http-fields/http-fields.xhtml

The `400` response has a `Content-Type: application/json` header and a body containing details about the error:

```json
{
  "error": {
    "code": "<error code>",
    "message": "<message>"
  }
}
```

`AwaitingDIDResolutionTryLater` means the service does not have an up-to-date DID document of the DID referenced in the Signed Claims but is performing or will perform a DID resolution after which the client may retry the request. The response may contain the HTTP header `Retry-After` to inform the client about the expected wait time.

`InvalidInput` means either the Signed Claims message is syntactically malformed, violates the signing profile (e.g. signing algorithm), or has an invalid signature relative to the currently resolved DID document.

### Retrieve Registration Receipt

#### Request

~~~
GET <Base URL>/entries/<Transaction ID>/receipt
~~~

#### Response

One of the following:

- HTTP Status `200` - Registration was successful and the Receipt is returned.
- HTTP Status `400` - Transaction exists but does not correspond to a Registration Request.
  - Error code `TransactionMismatch`
- HTTP Status `404` - Transaction is pending, unknown, or invalid.
  - Error code `TransactionPendingOrUnknown`
  - Error code `TransactionInvalid`

The `200` response contains the SCITT_Receipt in the body.

The `400` and `404` responses return the error details as described earlier.

The retrieved Receipt may be embedded in the corresponding COSE_Sign1 document in the unprotected header, see TBD.

[TODO] There's also the `GET <Base URL>/entries/<Transaction ID>` endpoint which returns the submitted COSE_Sign1 with the Receipt already embedded. Is this useful?


# Privacy Considerations

Unless advertised by the TS, every Issuer should treat its Claims as public. In particular, their Envelope and Statement should not carry any private information in plaintext.

# Security Considerations

On its own, verifying a Transparent Claim does not guarantee that its Envelope or contents are trustworthy---just that they have been signed by the apparent Issuer and counter-signed by the
TS. If the Verifier trusts the Issuer, it can infer that the Claim was issued with this Envelope and contents, which may be interpreted as the Issuer saying the Artifact is fit for its intended purpose. If the Verifier trusts the TS, it can independently infer that the Claim passed the TS Registration policy and that has been persisted in the Ledger. Unless advertised in the TS Registration policy, the Verifier should not assume that the ordering of Transparent Claims in the Ledger matches the ordering of their issuance.

Similarly, the fact that an Issuer can be held accountable for its Transparent Claims does not on its own provide any mitigation or remediation mechanism in case one of these Claims turned out to be misleading or malicious---just that signed evidence will be available to support them.

Issuers SHOULD ensure that the Statements in their Claims are correct and unambiguous, for example by avoiding ill-defined or ambiguous formats that may cause Verifiers to interpret the Claim as valid for some other purpose.

Issuers and Transparency Services SHOULD carefully protect their private signing keys
and avoid these keys for any purpose not described in this architecture. In case key re-use is unavoidable, they MUST NOT sign any other message that may be verified as an Envelope.

# IANA Considerations

See Body {{mybody}}.

--- back

# Attic

Not ready to throw these texts into the trash bin yet.
