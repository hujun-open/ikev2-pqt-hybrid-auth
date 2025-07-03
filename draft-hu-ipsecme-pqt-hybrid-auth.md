---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started. 
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Post-Quantum Traditional (PQ/T) Hybrid PKI Authentication in the Internet Key Exchange Version 2 (IKEv2)"
abbrev: "IKEv2 PQTH Auth"
category: std

docname: draft-hu-ipsecme-pqt-hybrid-auth-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: sec
workgroup: ipsecme
keyword:
 - Post-Quantum
 - Hybrid Authentication
 - IKEv2
venue:
  group: WG
  type: Working Group
  mail: ipsec@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/ipsec/
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Hu, Jun
    organization: Nokia
    email: jun.hu@nokia.com
    country: United States of America
 -
    fullname: Yasufumi Morioka
    organization: NTT DOCOMO, INC.
    email: yasufumi.morioka.dt@nttdocomo.com
    country: Japan
 -
    fullname: Wang, Guilin
    organization: Huawei
    email: Wang.Guilin@huawei.com
    country: Singapore
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "k.tirumaleswar_reddy@nokia.com"



normative:
  I-D.ietf-lamps-pq-composite-sigs:
  I-D.ietf-pquip-hybrid-signature-spectrums:
  I-D.ietf-lamps-cert-binding-for-multi-auth:
  I-D.ietf-lamps-dilithium-certificates:
  I-D.ietf-pquip-pqt-hybrid-terminology:
  RFC7296:
  RFC7427:
  RFC9593:
  X.690:
    title: "Information Technology - ASN.1 encoding rules: Specification of Basic Encoding Rules (BER), Canonical Encoding Rules (CER) and Distinguished Encoding Rules (DER)"
    seriesinfo:
      ISO/IEC: 8825-1:2021 (E)
      ITU-T: Recommendation X.690
    date: Feb.2021

informative:

  ML-DSA:
    title: Module-Lattice-Based Digital Signature Standard
    date: Aug.2023
    seriesinfo:
      NIST: FIPS-204
      State: Initial Public Draft
    target: https://csrc.nist.gov/pubs/fips/204/ipd
  RFC8784:
  RFC9370:


--- abstract

A Cryptographically Relevant Quantum Computer (CRQC) can break traditional public-key algorithms (e.g., RSA, ECDSA), which are typically used for authentication in IKEv2. Combining the post-quantum ML-DSA signature algorithm with a traditional signature algorithm provides protection against potential weaknesses or implementation flaws in ML-DSA. This draft defines hybrid PKI authentication methods for IKEv2 that ensure an attacker would need to break both algorithms to compromise the IKEv2 session.

--- middle

# Change in -02

* clarify the approach in the document is general
* dropping support for PreHash ML-DSA, change example to Pure Signature ML-DSA
* adding more details in signing process to align with ietf-lamps-pq-composite-sigs-04
* add text in Security Considerations to emphasize prohibit of key reuse
* clarify the both C and S bit MAY be 1 at the same time
* clarify the receiver behavior when the announcement contains no algid
* typo fixes

# Changes in -01

* Only use SUPPORTED_AUTH_METHODS for algorithm combination announcement, no longer use SIGNATURE_HASH_ALGORITHMS
* add flag field in the announcement
* clarify two types of PKI setup
* add some clarifications on how AUTH payload is computed

# Introduction

A Cryptographically Relevant Quantum Computer (CRQC) could break traditional asymmetric cryptographic algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. New Post-Quantum Cryptographic (PQC) algorithms for digital signature were recently published like ML-DSA {{ML-DSA}}, however by considering potential flaws in the new algorithm's specifications and implementations, it will take time for these new PQC algorithms to be field proven. So it is risky to only use PQC algorithms before they are mature. There is more detailed discussion on motivation of a hybrid approach for authentication in {{Section 1.3 of I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document describes an IKEv2 hybrid authentication scheme that contains both traditional and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.

The approach in this document could be a general framework that for all PQC and traditional algorithms, the combinations of ML-DSA variants and traditional algorithms are considered as instantiations of the general framework.

Following two types of setup are covered in the draft:

1. Composit certificate: A single certificate containing a composite key and signature, as defined in {{I-D.ietf-lamps-pq-composite-sigs}}.

2. Dual certificates: one with a traditional algorithm key and one with a PQC algorithm key. This method exemplifies a PQ/T hybrid protocol with non-composite authentication, as defined in {{Section 4 of I-D.ietf-pquip-pqt-hybrid-terminology}}. In this approach, two single-algorithm certificate chains are used in parallel. Each chain follows the standard PKI structure, and both chains together provide hybrid assurance without modifying the X.509 certificate format. While this method does not produce a single composite  signature, it ensures that both certificate chains are presented in the IKE_AUTH exchange, validated according to standard PKIX rules, and that each corresponding signature is computed over the IKEv2 signed octets, as defined in Section 2.15 of {{RFC7296}}, thereby cryptographically binding both certificates to the IKE SA.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Cryptographically Relevant Quantum Computer (CRQC): A quantum computer that is capable of breaking real world cryptographic systems.

Post-Quantum Cryptographic (PQC) algorithms: Asymmetric Cryptographic  algorithms are thought to be secure against CRQC.

Traditional Cryptographic algorithms: Existing asymmetric Cryptographic  algorithms could be broken by CRQC, like RSA, ECDSA, etc.

# IKEv2 Key Exchange

There is no changes introduced in this document to the IKEv2 key exchange process, although it MUST be also resilient to CRQC when using along with the PQ/T hybrid authentication, for example key exchange using the PPK as defined in {{RFC8784}}, or hybrid key exchanges that includes PQC algorithm via multiple key exchange process as defined in {{RFC9370}}.

# Exchanges

The hybrid authentication exchanges is illustrated in an example depicted in {{hybrid-auth-figure}}, using PPK as defined in {{RFC8784}} during key exchange, however it could be other key exchanges that involves PQC algorithm since how key exchange is done is transparent to authentication.

~~~~~~~~~~~
Initiator                         Responder
-------------------------------------------------------------------
HDR, SAi1, KEi, Ni,
          N(USE_PPK) -->
                  <--  HDR, SAr1, KEr, Nr, [CERTREQ,] N(USE_PPK),
                                      N(SUPPORTED_AUTH_METHODS)

HDR, SK {IDi, CERT+, [CERTREQ,]
        [IDr,] AUTH, SAi2,
        TSi, TSr, N(PPK_IDENTITY, PPK_ID),
        N(SUPPORTED_AUTH_METHODS)} -->
                            <--  HDR, SK {IDr, CERT+, [CERTREQ,]
                                      AUTH, [N(PPK_IDENTITY)]}
------------------------------------------------------------------- 
~~~~~~~~~~~
{: #hybrid-auth-figure title="Hybrid Authentication Exchanges with RFC8784 Key Exchange"}

# Composite Certificate

This draft extends and complements {{!PQC-AUTH=I-D.ipsecme-ikev2-pqc-auth}} which defines how to use Post-Quantum Cryptographic (PQC) signature algorithms (such as ML-DSA and SLH-DSA) in IKEv2 authentication. Both drafts share the same overarching goal: 

Enable IKEv2 to authenticate peers using PQC signature algorithms, ensuring security against quantum-capable adversaries.

Whereas {{PQC-AUTH}} specifies PQC-only authentication, this draft specifies how to deploy PQC and traditional algorithms together to provide hybrid assurance during the migration phase.

Both drafts:

* Do not require any changes to IKEv2 base protocol messages.
* Rely on the standard IKEv2 AUTH payload format {{RFC7296}}.
* Use SUPPORTED_AUTH_METHODS ({{RFC9593}}).

IKEv2 can use arbitrary signature algorithms as described in {{RFC7427}}, where the "Digital Signature" authentication method replaces older signature authentication methods. Both standalone PQC signature algorithms and composite signature algorithms can be incorporated using the "Signature Algorithm" field in the AUTH payload, as defined in {{!RFC7427}}.

For composite signatures, a single AlgorithmIdentifier describes a composite public key and a composite signature that combines multiple constituent algorithms (e.g., a traditional and a PQC algorithm) in accordance with {{I-D.ietf-lamps-pq-composite-sigs}}. This allows a single certificate and AUTH payload to provide hybrid assurance without requiring multiple exchanges.

AlgorithmIdentifier ASN.1 objects are used to uniquely identify  composite schemes, including the full parameter set for each constituent algorithm. This ensures unambiguous selection and verification of composite signature during authentication.

The signature MUST be computed and verified as specified in Section 2.15 of {{RFC7296}}. The Composite-ML-DSA.Sign function, defined in {{I-D.ietf-lamps-pq-composite-sigs}}, will be used by the sender to compute the signature field of the IKEv2 AUTH payload. Conversely, the Composite-ML-DSA.Verify function, also defined in
{{I-D.ietf-lamps-pq-composite-sigs}}, will be used by the receiver to verify the signature field of the AUTH payload.

# Dual Certificate Hybrid Authentication

This section describes how this draft leverages the mechanisms defined in {{?RFC4739}} to enable PQ/T hybrid authentication in IKEv2.

When using dual certificates, each peer performs multiple rounds of authentication as specified in {{?RFC4739}}:

* During capability negotiation, each peer indicates support for multiple authentications by including the MULTIPLE_AUTH_SUPPORTED notification in the initial key exchange.
* During the first IKE_AUTH exchange, the ANOTHER_AUTH_FOLLOWS notification is included to indicate that a subsequent authentication round will follow.

The authentication process is as follows:

1. First IKE_AUTH exchange
   - Uses the traditional certificate and signature.
   - Includes the ANOTHER_AUTH_FOLLOWS notification to signal that another authentication will occur.

2. Second IKE_AUTH exchange
   - Uses the PQC certificate and signature.
   - This completes the dual certificate authentication process.

Both authentication exchanges follow the standard IKEv2 signing procedure: each signature covers the signed octets defined in Section 2.15 of {{RFC7296}}, ensuring that both authentications are cryptographically bound to the same IKE SA.

Each party validates both authentication rounds. If either round fails, the IKE SA negotiation must fail.

## Example Flow

- IKE_SA_INIT: ECDH exchange, MULTIPLE_AUTH_SUPPORTED  
- IKE_SA_INTERMEDIATE: ML-KEM exchange  
- First IKE_AUTH: traditional CERT, traditional AUTH, ANOTHER_AUTH_FOLLOWS
- Second IKE_AUTH: PQC CERT, PQC AUTH

# Comparison of Composite and Dual Certificate Approaches

This section summarizes the advantages and disadvantages of using composite certificates and dual certificates for achieving hybrid post-quantum and traditional authentication.

## Composite Certificate

Advantages:

- A single certificate chain is used for both classical and post-quantum keys, simplifying certificate management.
- A single composite signature, rooted in one intermediate certificate chain, reduces protocol message size compared to transmitting multiple separate signatures, each of which would require its own certificate chain.
- No need to manage or validate multiple parallel certificate chains.
- Provides an integrated hybrid assurance model within a single certificate.
- No additional round-trip times (RTTs) are introduced.
- No changes to the base protocol are required to support the composite signature approach.

Disadvantages:

- Requires endpoints, relying parties and CAs to support composite public keys and composite signature verification, which may not yet be widely deployed.
- Introduces new certificate formats and verification logic that will need updates to PKI.
- It involves three transition paths, traditional only, composite, and PQC only, whereas the dual certificate approach requires only two (traditional and PQC).


## Dual Certificates

Advantages:

- Uses standard, single-algorithm X.509 certificates and chains, maximizing compatibility with existing PKI infrastructures.
- Maintains clear separation between traditional and post-quantum keys.
- No changes to certificate validation logic.

Disadvantages:

- Increases protocol message size due to the transmission of multiple certificate chains and signatures.
- Requires management of multiple certificates.
- Requires support of {{?RFC4739}}.

# Security Considerations

The security of general PQ/T hybrid authentication is discussed in {{I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document uses mechanisms defined in {{I-D.ietf-lamps-pq-composite-sigs}}, {{RFC7427}} and {{RFC9593}}, the security discussion in the corresponding RFCs also apply.

Ed25519 and Ed448 provide strong SUF (Strong Unforgeability under Forgery) security, which may remain secure even if ML-DSA were to be broken, at least until a CRQC becomes practical. IPsec deployments that prioritize SUF security may benefit from using Ed25519 or Ed448 in a composite signature together with ML-DSA. This mitigates the risk of ML-DSA compromise while retaining post-quantum protection and strong classical security guarantees.

One important security consideration mentioned in {{I-D.ietf-lamps-pq-composite-sigs}} worth repeating here is that component key used in either composite certificate MUST NOT be reused in any other cases including single-algorithm case.

Policy enforcement regarding the use of composite or dual certificates in IKEv2 is governed by the security policy of the authenticating peer. In typical deployments, the IPsec client and server are managed by the same organization or administrative domain, which ensures consistent policy configuration on both ends. When composite or dual certificate authentication is required by local policy, for example, during post-quantum migration the authenticating endpoint MUST reject an IKE_SA negotiation if only a single certificate or single signature is presented.

# IANA Considerations

None.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
