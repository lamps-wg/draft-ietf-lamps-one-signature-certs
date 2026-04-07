---
title: "One Signature Certificates"
abbrev: "OSC"
ipr: "trust200902"
category: std

docname: draft-lamps-one-signature-certs-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
date: 2026-02-26
consensus: true
v: 3
# area: Security
wg: LAMPS
keyword:
 - certificate
 - signature
 - X.509
venue:
  github: "lamps-wg/draft-ietf-lamps-one-signature-certs"

author:
  -
    ins: S. Santesson
    name: Stefan Santesson
    org: IDsec Solutions AB
    abbrev: IDsec Solutions
    street: Forskningsbyn Ideon
    city: Lund
    code: "223 70"
    country: SE
    email: sts@aaa-sec.com
  -
    name: Russ Housley
    ins: R. Housley
    org: Vigil Security, LLC
    abbrev: Vigil Security
    city: Herndon, VA
    country: US
    email: housley@vigilsec.com

normative:
  RFC2119:
  RFC3647:
  RFC5035:
  RFC5280:
  RFC5652:
  RFC7515:
  RFC8152:
  RFC8174:
  RFC9608:
  XMLDSIG11:
    title: "XML Signature Syntax and Processing Version 1.1"
    author:
      -
        ins: D. Eastlake
        name: Donald Eastlake
      -
        ins: J. Reagle
        name: Joseph Reagle
      -
        ins: D. Solo
        name: David Solo
      -
        ins: F. Hirsch
        name: Frederick Hirsch
      -
        ins: M. Nystrom
        name: Magnus Nystrom
      -
        ins: T. Roessler
        name: Thomas Roessler
      -
        ins: K. Yiu
        name: Kelvin Yiu
    date: 2013-04-11
    seriesinfo:
      "W3C": "Proposed Recommendation"
  ISOPDF2:
    title: "Document management -- Portable document format -- Part 2: PDF 2.0"
    author:
      org: ISO
    date: 2017-07
    seriesinfo:
      "ISO": "32000-2"
  XADES:
    title: "Electronic Signatures and Infrastructures (ESI); XAdES digital signatures; Part 1: Building blocks and XAdES baseline signatures"
    author:
      org: ETSI
    date: 2024-07
    seriesinfo:
      "ETSI": "EN 319 132-1 v1.3.1"
  PADES:
    title: "Electronic Signatures and Infrastructures (ESI); PAdES digital signatures; Part 1: Building blocks and PAdES baseline signatures"
    author:
      org: ETSI
    date: 2024-01
    seriesinfo:
      "ETSI": "EN 319 142-1 v1.2.1"
  CADES:
    title: "Electronic Signatures and Infrastructures (ESI); CAdES digital signatures; Part 1: Building blocks and CAdES baseline signatures"
    author:
      org: ETSI
    date: 2021-10
    seriesinfo:
      "ETSI": "EN 319 122-1 v1.2.1"

informative:
  RFC9321:

--- abstract

This document defines a profile for certificates that are issued for validation of the digital signature produced by a single signing operation. Each certificate is created at the time of signing and bound to the signed content. The associated signing key is generated, used to produce a single digital signature, and then immediately destroyed. The certificate never expires and is never revoked, which simplifies long-term validation.

--- middle

# Introduction

The landscape of server-based signing services has changed over the decades. Recently, one type of signature service has gained favor, where the signing private key and the signing certificate are created for each digital signature, rather than re-using a static key and certificate over an extended time period.

Some reasons why this type of signature services has been successful are:

- The certificate will always have a predictable validity time from the time of signing;
- The time of signing is guaranteed by the certificate issue date;
- The identity information in the certificate can be adapted to the signing context;
- Revocation of signing certificates is practically non-existent despite many years of operation and millions of signatures; and
- The signature service holds no pre-stored user keys or certificates.

While this type of signature service solves many problems, it still suffers from the complexity caused by expiring signing certificates. One solution to this problem is the Signature Validation Token (SVT) {{RFC9321}}, where future validation can rely on a previous successful validation rather than validation based on aging data.

This document takes this one step further and allows validation at any time in the future as long as trust in the CA certificate can be established.

## Basic features

One signature certificates have the following common characteristics:

- They never expire;
- They are never revoked;
- They are bound to a specific document content; and
- They assert that the corresponding private key was destroyed immediately after signing.

### Revocation

Traditional certificates that are re-used over time have many legitimate reasons for revocation, such as if the private key is lost or compromised. This can lead to large volumes of revocation data.

The fact that the same key is used many times exposes the key for the risks of loss, unauthorized usage, or theft. When many objects are signed with the same private key, the risk of exposure and the number of affected signed documents upon revocation increases, unless properly timestamped and properly verified.

When a signing key is used only once, that risk of exposure is greatly reduced, and it has been shown that most usages of dedicated private keys and certificates no longer require revocation.

The CA can readily attest that a certain procedure was followed when the certificate was issued. As a matter of policy, the certificate itself is an attestation that the CP and CPS {{RFC3647}} were followed successfully when the signature was created. Certificates issued according to this profile therefore only attest to the validity at the time of issuance and signing, rather than a retroactive state at the time of validation. This profile is intended for those applications where this declaration of validity is relevant and useful.

Applications that require traditional revocation checking that provides the state at the time of validation MUST NOT use this profile.

An example usage where this is useful is in services where the signed document is stored as an internal evidence record, such as when a Tax agency allows citizens to sign their tax declarations. This record is then pulled out and used only in case of a dispute where the identified signer challenges the signature. A revocation service is less likely to contribute to this process. If the challenge is successful, the signed document will be removed without affecting any other signed documents in the archive.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Certificate content

Conforming certificates SHALL meet all requirements of this section.

Certificates MUST indicate that a certificate has no well-defined expiration date by setting the notAfter field to the GeneralizedTime value 99991231235959Z, as defined in {{RFC5280}}.

Certificates MUST include the id-ce-noRevAvail extension in compliance with {{RFC9608}}, indicating that this certificate is not supported by any revocation mechanism.

Certificates MUST include the signedDocumentBinding extension, binding the certificate to a specific signed content.

## The signedDocumentBinding extension

The signedDocumentBinding extension binds a certificate to a specific signed content. When present, conforming CAs SHOULD mark this extension as non-critical.


    name           id-pe-signedDocumentBinding
    OID            { id-pe TBD }
    syntax         SignedDocumentBinding
    criticality    SHOULD be FALSE

    SignedDocumentBinding ::= SEQUENCE {
    dataTbsHash     OCTET STRING,
    hashAlg         DigestAlgorithmIdentifier,
    bindingType     UTF8String OPTIONAL }


The dataTbsHash field MUST contain a hash of the data to be signed.

The hashAlg field MUST contain the AlgorithmIdentifier of the hash algorithm used to generate the dataTbsHash value.

The bindingType field MAY contain an identifier that specifies how the data to be signed is derived from the digital object to be signed.

Adding this extension to a certificate is a statement by the CA that the signing key is generated exclusively for the purpose of signing the document bound by this extension, and that the signing key is destroyed after signing. The details for this procedure and how the destruction of the signing key is assured SHOULD be outlined in the certificate policy {{RFC3647}} of the issued certificate.

## Defined bindingType identifiers

The bindingType field defines how the data to be signed (dataTbsHash) is derived from the signed document.
This field identifies a deterministic procedure for selecting the portion of the signed content that is included in the hash computation.
When the field is omitted, the rules for the default binding type apply.

The purpose of the dataTbsHash value is to bind the certificate to the document being signed in order to prevent re-use of the signing key for multiple signed documents. This enforces the contract that the signing key is used only once for creation of one signature only. Validators SHOULD verify that the signed document matches the certificate’s binding information.
This verification is not required for the signature to validate successfully but provides an additional safeguard against misuse or substitution of certificates.

This document defines a set of bindingType identifiers. Additional bindingType identifiers MAY be defined by future specifications.

### Default Binding

When the bindingType is absent, the default binding applies.
In this case, the dataTbsHash value is the hash of the exact data that is hashed and signed by the signature format in use.

Examples include:
- For XML Signatures {{XMLDSIG11}}, the hash of the SignedInfo element.
- For CMS Signatures {{RFC5652}}, the DER-encoded SignedAttributes structure.
- For other formats, the data structure input directly to the signature algorithm.

This bindingType MUST NOT be used when the data to be signed includes either the signer certificate itself or a hash of the signer certificate. This includes JWS and COSE signed documents that can include signer certificates in the protected header. JWS signatures {{RFC7515}} MUST use the "jws" bindingType and COSE signatures {{RFC8152}} MUST use the "cose" binding type.

### CAdES Binding

Identifier: "cades"

For CMS {{RFC5652}} or ETSI CAdES {{CADES}} signatures incorporating SigningCertificate or SigningCertificateV2 attributes {{RFC5035}} in signedAttrs,
the dataTbsHash value is computed over the DER encoding of SignerInfo excluding any instances of SigningCertificate or SigningCertificateV2 attributes from the SignedAttributes set.

This bindingType also applies to PDF {{ISOPDF2}} and ETSI PAdES {{PADES}} signed documents when applicable due to its use of CMS for signing.

### XAdES Binding

Identifier: "xades"

For ETSI XML Advanced Electronic Signatures {{XADES}}, the dataTbsHash value is computed over the canonicalized SignedInfo element,
with any Reference elements whose Type attribute equals "http://uri.etsi.org/01903#SignedProperties" removed prior to hashing.
This ensures that the SignedProperties element, which may contain references to the signing certificate, does not create a circular dependency. Extraction of the Reference element MUST be done by removing only the characters from the leading &lt;Reference&gt; tag up to and including the ending &lt;/Reference&gt; tag, preserving all other bytes of SignedInfo unchanged, including any white space or line feeds.

Note: This operation is purely textual and does not require XML parsing beyond locating the tag boundaries.

### JWS Binding

Identifier: "jws"

For JSON Web Signatures (JWS) {{RFC7515}}, the dataTbsHash value is computed over the payload only.
The protected header and any unprotected header parameters MUST NOT be included in the hash calculation.

This exclusion avoids circular dependencies where certificate data may appear in the protected header.

### COSE Binding

Identifier: "cose"

For COSE signatures {{RFC8152}}, the dataTbsHash value is computed over the payload only.
The protected header and any unprotected header parameters MUST NOT be included in the hash calculation.

This exclusion avoids circular dependencies where certificate data may appear in the protected header.

# ASN.1 Module

    <CODE BEGINS>
       SignedDocumentBindingExtn
         { iso(1) identified-organization(3) dod(6) internet(1)
           security(5) mechanisms(5) pkix(7) id-mod(0)
           id-mod-signedDocumentBinding(TBD) }

       DEFINITIONS IMPLICIT TAGS ::=
       BEGIN

       IMPORTS
         EXTENSION, id-pkix, id-pe
         FROM PKIX-CommonTypes-2009  -- RFC 5912
           { iso(1) identified-organization(3) dod(6) internet(1)
             security(5) mechanisms(5) pkix(7) id-mod(0)
             id-mod-pkixCommon-02(57) }

         DigestAlgorithmIdentifier
         FROM CryptographicMessageSyntax-2010 -- RFC 6268
           { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
             pkcs-9(9) smime(16) modules(0) id-mod-cms-2009(58) } ;

       -- signedDocumentBinding Certificate Extension

       ext-SignedDocumentBinding EXTENSION ::= {
         SYNTAX SignedDocumentBinding
         IDENTIFIED BY id-pe-signedDocumentBinding }

       SignedDocumentBinding ::= SEQUENCE {
         dataTbsHash     OCTET STRING,
         hashAlg         DigestAlgorithmIdentifier,
         bindingType     UTF8String OPTIONAL }

       -- signedDocumentBinding Certificate Extension OID

       id-pe-signedDocumentBinding OBJECT IDENTIFIER ::= { id-pe TBD }

       END
     <CODE ENDS>

# Security Considerations

## Certificates Without Revocation

Certificates conforming to this profile include the id-ce-noRevAvail extension and therefore do not provide any revocation mechanism. Such certificates attest only to the state of trust and correctness of procedures at the time of issuance.

The Security considerations in {{RFC9608}} also applies to this document.

## Signed Document Binding

The signedDocumentBinding extension binds the certificate to specific signed content by including a hash of the data to be signed. Verification of this binding is not required for successful cryptographic validation of the signature. A signature can therefore validate correctly even if the binding is not checked.

However, a relying party SHOULD verify that the signed content matches the dataTbsHash value in the signedDocumentBinding extension. Performing this check ensures that the certificate is used only with the content for which it was issued and enforces the intended scope of the certificate.

The security model of this profile states that the associated private key is generated for, and used in, exactly one signing operation and is then destroyed. This property holds independently of whether the binding is verified by the relying party. Nevertheless, failure to verify the binding weakens the protections provided by this profile and increases the risk of certificate substitution or unintended certificate reuse.

When verified, the signedDocumentBinding extension provides an additional safeguard against the use of the certificate for any signature other than the one for which it was issued.

# IANA Considerations

## Registry for signedDocumentBinding bindingType Identifiers

IANA is requested to create a new registry entitled: “Signed Document Binding Type Identifiers”

This registry shall contain identifiers used in the bindingType field of the signedDocumentBinding certificate extension defined in this document.

### Registry Contents

Each registry entry shall contain the following fields:

- Identifier: A UTF-8 string identifying the binding type.
- Description: A brief description of how the dataTbsHash value is computed.
- Reference: A reference to the document that defines the binding type.

### Registration Policy

The registration policy for this registry is Specification Required as defined in {{RFC8174}}.

The designated expert(s) SHALL ensure that:

- The binding type definition clearly specifies a deterministic and unambiguous procedure for computing the dataTbsHash value.
- The specification explains how circular dependencies with certificate inclusion are avoided, where applicable.
- The identifier is unique within the registry.

### Initial Registry Contents

IANA is requested to populate the registry with the following initial values:

- Identifier: (absent)
- Description: Default binding as defined in this document
- Reference: This document

- Identifier: cades
- Description: CMS/CAdES binding excluding SigningCertificate attributes
- Reference: This document

- Identifier: xades
- Description: XAdES binding excluding SignedProperties reference
- Reference: This document

- Identifier: jws
- Description: JWS payload-only binding
- Reference: This document

- Identifier: cose
- Description: COSE payload-only binding
- Reference: This document

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
