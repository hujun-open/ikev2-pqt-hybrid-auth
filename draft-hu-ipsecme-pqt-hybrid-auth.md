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
    email: "kondtir@gmail.com"

normative:
  I-D.ietf-lamps-pq-composite-sigs:
  I-D.ietf-pquip-hybrid-signature-spectrums:
  I-D.ietf-lamps-cert-binding-for-multi-auth:
  I-D.ietf-lamps-dilithium-certificates:
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

 One IPsec area that would be impacted by Cryptographically Relevant Quantum Computer (CRQC) is IKEv2 authentication based on traditional asymmetric cryptographic algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. There are new Post-Quantum Cryptographic (PQC) algorithms for digital signature like NIST {{ML-DSA}}, however it takes time for new cryptographic algorithms to mature, so there is security risk to use only the new algorithm before it is field proven. This document describes a IKEv2 hybrid authentication scheme that could contain both traditional and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.


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

A Cryptographically Relevant Quantum Computer (CRQC) could break traditional asymmetric cryptographic algorithms: e.g RSA, ECDSA; which are widely deployed authentication options of IKEv2. New Post-Quantum Cryptographic (PQC) algorithms for digital signature were recently published like NIST {{ML-DSA}}, however by considering potential flaws in the new algorithm's specifications and implementations, it will take time for these new PQC algorithms to be field proven. So it is risky to only use PQC algorithms before they are mature. There is more detailed discussion on motivation of a hybrid approach for authentication in {{Section 1.3 of I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document describes an IKEv2 hybrid authentication scheme that contains both traditional and PQC algorithms, so that authentication is secure as long as one algorithm in the hybrid scheme is secure.

Each IPsec peer announces the support of hybrid authentication via SUPPORTED_AUTH_METHODS notification as defined in {{RFC9593}}, generates and verifies AUTH payload using composite signature like the procedures defined in {{I-D.ietf-lamps-pq-composite-sigs}}.

The approach in this document could be a general framework that for all PQC and traditional algorithms, the combinations of ML-DSA variants and traditional algorithms are considered as instantiations of the general framework.

Following two types of setup are covered:

1. Type-1: A single certificate that has composite key as defined in {{I-D.ietf-lamps-pq-composite-sigs}}
2. Type-2: Two certificates, one with traditional algorithm key and one with PQC algorithm key


# Conventions and Definitions

{::boilerplate bcp14-tagged}

Cryptographically Relevant Quantum Computer (CRQC): A quantum computer that is capable of breaking real world cryptographic systems.

Post-Quantum Cryptographic (PQC) algorithms: Asymmetric Cryptographic  algorithms are thought to be secure against CRQC.

Traditional Cryptographic algorithms: Existing asymmetric Cryptographic  algorithms could be broken by CRQC, like RSA, ECDSA ..etc.

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
~~~~~~~~~~~
{: #hybrid-auth-figure title="Hybrid Authentication Exchanges with RFC8784 Key Exchange"}

## Announcement

Announcement of support hybrid authentication is through SUPPORTED_AUTH_METHODS notification as defined in {{RFC9593}}, which includes a list of acceptable authentication methods announcements. this document defines a hybrid authentication announcements with following format:


                         1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Length (>=2) |  Auth Method  |   Cert Link 1 | Alg 1 flag    |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Alg 1 Len     |                                               |
    +-+-+-+-+-+-+-+-+                                               |
    ~                      AlgorithmIdentifier 1                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Cert Link 2   | Alg 2 flag    |  Alg 2 Len    |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~                      AlgorithmIdentifier 2                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    ~                      ...                                      ~
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    | Cert Link 3   | Alg 3 flag    |  Alg 3 Len    |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               +
    |                                                               |
    ~                      AlgorithmIdentifier N                    ~
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{: #ds-announce title="Hybrid Authentication Announcement"}

The announcement includes a list of N algorithms could be used for hybrid signature

* Auth Method: A new value to be allocated by IANA
* Cert Link N: Links corresponding signature algorithm N with a particular CA. as defined in {{Section 3.2.2 of RFC9593}}
* Alg N Flag:
  * C: set to 1 if the algorithm could be used in type-1 setup
  * S: set to 1 if the algorithm could be used in type-2 setup
  * Both C and S MAY be set to 1 but MUST NOT set to zero at the same time
  * RESERVED: set to 0

~~~~~~~~~~~
     0 1 2 3 4 5 6 7
    +-+-+-+-+-+-+-+-+
    |C|S| RESERVED  |
    +-+-+-+-+-+-+-+-+
~~~~~~~~~~~
{: #announce-flag title="Algorithm Flag"}

* AlgorithmIdentifier N: The variable-length ASN.1 object that is encoded using Distinguished Encoding Rules (DER) {{X.690}} and identifies the  algorithm of a composite signature as defined in {{Section 7 of I-D.ietf-lamps-pq-composite-sigs}}.


### Sending Announcement

As defined in {{RFC9593}}, responder includes SUPPORTED_AUTH_METHODS in IKE_SA_INIT response (and potentially also in IKE_INTERMEDIATE response), while initiator includes the notification in IKE_AUTH request.

Sender includes a hybrid authentication announcement in SUPPORTED_AUTH_METHODS, which contains 0 or N composite signature AlgorithmIdentifiers sender accepts, each AlgorithmIdentifier identifies a combination of algorithms:

* a traditional PKI algorithm with corresponding hash algorithm (e.g. id-RSASA-PSS with id-sha256)
* a PQC algorithm (e.g. id-ML-DSA-44)
  * in case of Hash ML-DSA, there is also a pre-hash algorithm (e.g. id-sha256)

In case of type-2 setup, even though the certificate is not composite key certificate, system still uses a composite signature algorithm that corresponds to the combination of two certificates PKI algorithms and hash algorithm(s).

C and S bits in flag field are set according to whether sender accepts the algorithm combination in type-1/type-2 setup.

Announcement without any AlgorithmIdentifiers signals that there is no particular restrictions on algorithm.

### Receiving Announcement

If hybrid authentication announcement is received, and receiver chooses to authenticate itself using hybrid authentication, then based on its local policy and certificates, one AlgorithmIdentifier (which identifies a combination of algorithms) in the hybrid authentication announcement and a PKI setup (type-1 or type-2) is chosen to create its AUTH and CERT payload(s). If there is no AlgorithmIdentifier in the announcement, receiver MAY choose AlgorithmIdentifier just base on its local policy and certificates.




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

For hybrid authentication, the AUTH Method has value defined in {{announcement}}

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
2. Follow Sign operation identified by A, e.g. {{Section 4.2.1 of I-D.ietf-lamps-pq-composite-sigs}}. the ctx input is the string of "IKEv2-PQT-Hybrid-Auth". this step outputs the composite signature, a CompositeSignatureValue.
3. CompositeSignatureValue is serialized per {{Section 4.5 of I-D.ietf-lamps-pq-composite-sigs}}, the output is used as Signature Value in the Authentication Data field.

note: in case ML-DSA, only pure signature mode as defined in {{Section 4.2 of I-D.ietf-lamps-pq-composite-sigs}} is used, the PreHash ML-DSA mode MUST NOT be used, see {{Section 8.1 of I-D.ietf-lamps-dilithium-certificates}} for the rationale.

Following is an initiator example:

1. A is id-MLDSA44-RSA2048-PSS, which uses pure signature mode id-ML-DSA-44 and id-RSASSA-PSS with id-sha256
2. Follow {{Section 4.2.1 of I-D.ietf-lamps-pq-composite-sigs}} with following input:

    - sk is the private key of the signing composite key certificate
    - M is InitiatorSignedOctets
    - ctx is "IKEv2-PQT-Hybrid-Auth"


The signing composite certificate MUST be the first CERT payload.

### Type-2

The procedure is same as Type-1, use private key of traditional and PQC certificate accordingly; e.g. in Sign procedure define in {{Section 4.2.1 of I-D.ietf-lamps-pq-composite-sigs}}, the `mldsaSK` is the private key of ML-DSA certificate, while `tradSK` is the private key of traditional certificate.

With the example in {{type-1}}:

  - mldsaSK is the private key of ML-DSA certificate, tradSK is the private key of the RSA certificate
  - M is InitiatorSignedOctets
  - ctx is "IKEv2-PQT-Hybrid-Auth"

The signing PQC certificate MUST be the first CERT payload in the IKEv2 message, while traditional certificate MUST be the second CERT payload.



#### RelatedCertificate
In type-2 setup, the signing certificate MAY contain RelatedCertificate extension, then the receiver SHOULD verify the extension according to {{Section 4.2 of I-D.ietf-lamps-cert-binding-for-multi-auth}}, failed verification SHOULD fail authentication.

# Certificate Chain Delimitation for Type-2 Authentication

The IKEv2 protocol, as defined in {{RFC7296}}, assumes a single-algorithm authentication model. As explained in Section 3.6, the sender's AUTH payload is validated using the public key in the first CERT payload, while subsequent CERT payloads are treated as intermediate certificates in a single chain.

In contrast, Type-2 hybrid authentication requires the validation of two distinct signatures within the AUTH payload, each corresponding to a different cryptographic algorithm. This necessitates the exchange of two separate end-entity certificates and their respective chains. 

For Type-2 hybrid authentication, an IKE_AUTH message contains two distinct certificate chains, with each certificate sent in a separate CERT payload. Without a clear delimiter, concatenating the two chains would be ambiguous, as the receiving peer could not reliably identify which CERT payload contains the end-entity certificate for the second signature. To avoid this ambiguity and ensure interoperability, this document specifies a method to clearly delimit the two chains.

A new Notify Payload with the CERT_CHAIN_DELIMITER message type is used for this purpose. This is a status type notification with an empty Notification Data field.

The IKE_AUTH message structure for Type-2 hybrid authentication MUST follow this order:

~~~~~~~~~~~
IKE_AUTH -> IDi, AUTH, CERT*, CERT_CHAIN_DELIMITER, CERT*, [CP], [SA], [TSi, TSr]
~~~~~~~~~~~

* CERT (first instance): The series of CERT payloads for the first certificate chain (end-entity and intermediates).  
* CERT_CHAIN_DELIMITER: The Notify Payload delimiter.  
* CERT (second instance): The series of CERT payloads for the second certificate chain (end-entity and intermediates).  

In the CERT_CHAIN_DELIMITER Notify Payload: 

* Protocol-ID: MUST be set to 0, as this notification is not specific to any protocol (e.g., AH or ESP).  
* SPI Size: MUST be set to 0, indicating that no SPI field is included in this notification.  
* Notification Data : MUST be empty. 

Implementations MUST correctly process the delimiter to identify the separation of the two chains.

# Security Considerations

The security of general PQ/T hybrid authentication is discussed in {{I-D.ietf-pquip-hybrid-signature-spectrums}}.

This document uses mechanisms defined in {{I-D.ietf-lamps-pq-composite-sigs}}, {{RFC7427}} and {{RFC9593}}, the security discussion in the corresponding RFCs also apply.

One important security consideration mentioned in {{I-D.ietf-lamps-pq-composite-sigs}} worth repeating here is that component key used in either {{type-1}} or {{type-2}} MUST NOT be reused in any other cases including single-algorithm case.


# IANA Considerations

This document requests a value in "IKEv2 Authentication Method" subregistry under IANA "Internet Key Exchange Version 2 (IKEv2) Parameters" registry

IANA is requested to assign a new Notify Message Type from the "IKEv2 Notify Message Status Types" range for IKEv2, as follows:

## CERT_CHAIN_DELIMITER

* Notify Message Type Name: CERT_CHAIN_DELIMITER  
* Value: To be assigned by IANA from the Status Types range  

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
