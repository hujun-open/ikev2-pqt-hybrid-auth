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
    fullname: Jun Hu
    organization: Nokia
    email: jun.hu@nokia.com
    country: United States of America
 -
    fullname: Yasufumi Morioka
    organization: NTT DOCOMO, INC.
    email: yasufumi.morioka.dt@nttdocomo.com
    country: Japan
 -
    fullname: Guilin Wang
    organization: Huawei
    email: Wang.Guilin@huawei.com
    country: Singapore



normative:
  I-D.ietf-lamps-pq-composite-sigs:
  I-D.ietf-pquip-hybrid-signature-spectrums:
  RFC9763:
  RFC9881:
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
    target: https://csrc.nist.gov/pubs/fips/204/final


  ML-KEM:
    title: Module-Lattice-Based Key-Encapsulation Mechanism Standard
    date: Aug.2023
    seriesinfo:
      NIST: FIPS-203
    target: https://csrc.nist.gov/pubs/fips/203/final

  RFC8784:
  RFC9370:
  I-D.ietf-ipsecme-ikev2-mlkem:


--- abstract

 One IPsec area that would be impacted by Cryptographically Relevant Quantum Computer (CRQC) is IKEv2 authentication based on traditional asymmetric cryptographic algorithms: e.g RSA, ECDSA, which are widely deployed authentication options of IKEv2. There are new Post-Quantum Cryptographic (PQC) algorithms for digital signature like NIST {{ML-DSA}}, However, it takes time for new cryptographic algorithms to mature, There is security risk to use only the new algorithm before it is field proven. This document describes a hybrid PKI authentication scheme for IKEv2 that incorporates both traditional and PQC digital signature algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.


--- middle

# Change log

## changes in -04

* align to draft-ietf-lamps-pq-composite-sigs-14
* add text to clarify two setup types
* add text to describe the example exchange in section 5
* clarify using of pre-hash alg
* clarify sign operation in type-2
* ietf-lamps-cert-binding-for-multi-auth is now RFC9763
* ietf-lamps-dilithium-certificates is now RFC9881
* editorial changes


## changes in -03

* version bump to keep doc alive

## Changes in -02

* clarify the approach in the document is general
* dropping support for PreHash ML-DSA, change example to Pure Signature ML-DSA
* adding more details in signing process to align with ietf-lamps-pq-composite-sigs-04
* add text in Security Considerations to emphasize prohibit of key reuse
* clarify the both C and S bit MAY be 1 at the same time
* clarify the receiver behavior when the announcement contains no algid
* typo fixes

## Changes in -01

* Only use SUPPORTED_AUTH_METHODS for algorithm combination announcement, no longer use SIGNATURE_HASH_ALGORITHMS
* add flag field in the announcement
* clarify two types of PKI setup
* add some clarifications on how AUTH payload is computed



# Introduction

A Cryptographically Relevant Quantum Computer (CRQC) could break traditional asymmetric cryptographic algorithms: e.g RSA, ECDSA, which are widely deployed authentication options of IKEv2. New Post-Quantum Cryptographic (PQC) algorithms for digital signature were recently published like NIST {{ML-DSA}}, However, by considering potential flaws in the new algorithm's specifications and implementations, it will take time for these new PQC algorithms to be field proven. So it is risky to only use PQC algorithms before they are mature. There is more detailed discussion on motivation of a hybrid approach for authentication in {{Section 1.2 of I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document describes a post-quantum traditional (PQ/T) hybrid digital signature authentication scheme for IKEv2 that incorporates both traditional and PQC digital signature algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.

Each IPsec peer announces the support of hybrid authentication via SUPPORTED_AUTH_METHODS notification as defined in {{RFC9593}}, generates and verifies AUTH payload using composite signature using the procedures defined in {{I-D.ietf-lamps-pq-composite-sigs}}.

The approach specified in this document is a general framework for all PQC and traditional algorithms. The combinations of ML-DSA variants and traditional algorithms given in this document are instantiations of the general framework.

There are two types of PQ/T hybrid PKI setup:

1. Type-1: A single certificate that has a composite key as defined in {{I-D.ietf-lamps-pq-composite-sigs}}, which contains two component keys: one traditional key + one PQC key.
2. Type-2: Two certificates, one certificate with traditional algorithm key and one certificate with PQC algorithm key as described in {{RFC9763}}, Each certificate MAY contain RelatedCertificate extension to associate with the other certificate.

A given deployment could use either type to provide PQ/T hybrid PKI. This document supports both types.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Cryptographically Relevant Quantum Computer (CRQC): A quantum computer that is capable of breaking real world cryptographic systems.

Post-Quantum Cryptographic (PQC) algorithms: Asymmetric Cryptographic  algorithms are thought to be secure against CRQC.

Traditional Cryptographic algorithms: Existing asymmetric Cryptographic  algorithms could be broken by CRQC, like RSA, ECDSA ..etc.

# IKEv2 Key Exchange
There is no changes introduced in this document to the IKEv2 key exchange process, although it MUST be also resilient to CRQC when using along with the PQ/T hybrid authentication, for example key exchange using the PPK as defined in {{RFC8784}}, or hybrid key exchanges that includes PQC algorithm like {{ML-KEM}} via multiple key exchange process as defined in {{I-D.ietf-ipsecme-ikev2-mlkem}}.


# Exchanges

The hybrid authentication exchanges is illustrated in an example depicted in {{hybrid-auth-figure}}, using PPK as defined in {{RFC8784}} during key exchange. However, other PQC key exchanges could also be used since how key exchange is done is independent from authentication.

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
~~~~~~~~~~~
{: #hybrid-auth-figure title="Hybrid Authentication Exchanges with RFC8784 Key Exchange"}

1. Responder announces the hybrid authentication support via SUPPORTED_AUTH_METHODS notification in IKE_SA_INIT response message. The notification includes the combinations of PQC, traditional, hash algorithm and type of hybrid PKI setup that responder supports.

2. Initiator chooses a combination from responder's SUPPORTED_AUTH_METHODS, uses the combination to generate the AUTH payload, along with corresponding signing certificate(s) in CERT payload(s), and includes its support of hybrid combinations in SUPPORTED_AUTH_METHODS notification of IKE_AUTH request message.

3. Responder chooses a combination from initiator's SUPPORTED_AUTH_METHODS, uses the combination to generate the AUTH payload, and includes corresponding signing certificate(s) in CERT payload(s) of IKE_AUTH response message.

## Announcement
{: #announcement}

Announcement of support for hybrid authentication is through the SUPPORTED_AUTH_METHODS notification as defined in {{RFC9593}}, using multi-octet announcements. This document uses the existing multi-octet announcement format from {{RFC9593}} with the following AUTH_METHOD values:

1. For type-1 (composite key certificate): use AUTH_METHOD value 14 (Digital Signature, as defined in {{RFC7427}}) together with the composite signature AlgorithmIdentifier as defined in {{Section 7 of I-D.ietf-lamps-pq-composite-sigs}}.

2. For type-2 (two separate certificates): use a new IANA-assigned AUTH_METHOD value together with the composite signature AlgorithmIdentifier corresponding to the combination of the two certificates.

There is no change to the existing multi-octet announcement protocol format defined in {{RFC9593}}. The only new protocol element introduced by this document is the new IANA-assigned AUTH_METHOD value for type-2.

For example, if a system supports the following authentication configurations:

* A: MLDSA44 + RSA2048_PSS as type-1
* B: MLDSA44 + ECDSA-P256 as type-1
* C: MLDSA44 + RSA2048_PSS as type-2

It will include 3 multi-octet announcements in the SUPPORTED_AUTH_METHODS payload:

* Auth-method 14 with AlgorithmIdentifier id-MLDSA44-RSA2048-PSS-SHA256, for A above
* Auth-method 14 with AlgorithmIdentifier id-MLDSA44-ECDSA-P256-SHA256, for B above
* Auth-method NEW_VAL_for_TYPE2 with AlgorithmIdentifier id-MLDSA44-RSA2048-PSS-SHA256, for C above

~~~~~~~~~~~
                         1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  |      14       |   Cert Link   |               |  <- A
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~          id-MLDSA44-RSA2048-PSS-SHA256 (AlgorithmIdentifier)  ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  |      14       |   Cert Link   |               |  <- B
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~          id-MLDSA44-ECDSA-P256-SHA256 (AlgorithmIdentifier)   ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>3)  | NEW_VAL_TYPE2 |   Cert Link   |               |  <- C
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~          id-MLDSA44-RSA2048-PSS-SHA256 (AlgorithmIdentifier)  ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #sam-payload title="Example SUPPORTED_AUTH_METHODS Payload with 3 Announcements"}

Each AlgorithmIdentifier is the variable-length ASN.1 object encoded using Distinguished Encoding Rules (DER) {{X.690}} that identifies a composite signature algorithm as defined in {{Section 7 of I-D.ietf-lamps-pq-composite-sigs}}, specifying a combination of:

* a PQC algorithm (e.g. id-ML-DSA-44)
* a traditional PKI algorithm (e.g. id-RSASA-PSS)
* a pre-hash algorithm (e.g. id-sha256)




## AUTH & CERT payload

The IKEv2 AUTH payload has following format as defined in {{Section 3.8 of RFC7296}}:


                            1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | Next Payload  |C|  RESERVED   |         Payload Length        |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | Auth Method   |                RESERVED                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~                      Authentication Data                      ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #rfc7296-auth title="AUTH payload"}

For hybrid authentication, the Auth Method is either value 14 (Digital Signature) for type-1 or the new IANA-assigned value for type-2, as defined in {{announcement}}

The Authentication Data field follows format defined in {{Section 3 of RFC7427}}:



                           1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | ASN.1 Length  | AlgorithmIdentifier ASN.1 object              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~        AlgorithmIdentifier ASN.1 object continuing            ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                                                               |
      ~                         Signature Value                       ~
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #ha-auth-data title="Authentication Data in hybrid AUTH payload"}

Based on selected AlgorithmIdentifier and setup type, the Signature Value is created via procedure defined in {{type-1}}, {{type-2}}.


### Type-1
Assume selected AlgorithmIdentifier is A.

1. There is no change on data to be signed, e.g. InitiatorSignedOctets/ResponderSignedOctets as defined in {{Section 2.15 of RFC7296}}
2. Follow Sign operation identified by A, e.g. {{Section 3.2 of I-D.ietf-lamps-pq-composite-sigs}}. The ctx input is the string of "IKEv2-PQT-Hybrid-Auth". This step outputs the composite signature, a CompositeSignatureValue.
3. CompositeSignatureValue is serialized per {{Section 4.3 of I-D.ietf-lamps-pq-composite-sigs}}, and the output is used as Signature Value in the Authentication Data field.

Note: {{I-D.ietf-lamps-pq-composite-sigs}} uses a pre-hash algorithm with {{ML-DSA}} pure mode (Algorithm 2), not the HashML-DSA as defined in {{ML-DSA}}, see {{Section 2.1 of I-D.ietf-lamps-pq-composite-sigs}} for the rationale.

Following is an initiator example:

1. A is id-MLDSA44-RSA2048-PSS-SHA256, which uses PQC ML-DSA-44 and traditional RSASSA-PSS with pre-hash function SHA256
2. Follow {{Section 3.2 of I-D.ietf-lamps-pq-composite-sigs}} with following inputs:

    - sk is the private key of the signing composite key certificate
    - M is InitiatorSignedOctets
    - ctx is "IKEv2-PQT-Hybrid-Auth"


The signing composite certificate MUST be the first CERT payload.

### Type-2

1. Combine PQC key and traditional key into composite key using SerializePrivateKey operation as defined in {{Section 4.2 of I-D.ietf-lamps-pq-composite-sigs}}.
2. Follow Sign operation as {{type-1}}

Note: {{Section 6 of RFC9881}} defines 3 options for ML-DSA private key storage, this document requires options that include seed since Sign operation of {{I-D.ietf-lamps-pq-composite-sigs}} only supports seed.

With example in {{type-1}}:

  - sk is the combined private key, e.g. output of SerializePrivateKey
  - M is InitiatorSignedOctets
  - ctx is "IKEv2-PQT-Hybrid-Auth"

The signing PQC certificate MUST be the first CERT payload in the IKEv2 message, while traditional certificate MUST be the second CERT payload.



#### RelatedCertificate
In type-2 setup, the signing certificate MAY contain RelatedCertificate extension, then the receiver SHOULD verify the extension according to {{Section 4.2 of RFC9763}}. Failed verification SHOULD fail authentication.


# Security Considerations

The security of general PQ/T hybrid authentication is discussed in {{I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document uses mechanisms defined in {{I-D.ietf-lamps-pq-composite-sigs}}, {{RFC7427}} and {{RFC9593}}, so the security discussion in the corresponding RFCs also apply.

One important security consideration mentioned in {{I-D.ietf-lamps-pq-composite-sigs}} worth repeating here is that component key used in either {{type-1}} or {{type-2}} MUST NOT be reused in any other cases including single-algorithm case.


# IANA Considerations

This document requests a new value in the "IKEv2 Authentication Method" subregistry under the IANA "Internet Key Exchange Version 2 (IKEv2) Parameters" registry for the type-2 (two-certificate) PQ/T hybrid authentication method. Type-1 (composite key certificate) hybrid authentication reuses the existing AUTH_METHOD value 14 (Digital Signature) and requires no new IANA allocation.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
