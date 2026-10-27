---
title: "One Signature Certificates"
abbrev: "OSC"
ipr: "trust200902"
category: std
replaces: draft-santesson-one-signature-certs

docname: draft-ietf-lamps-one-signature-certs-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
date: 2026-07-01
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
  RFC5912:
  RFC6268:
  RFC7515:
  RFC8152:
  RFC8174:
  RFC9608:
  X680:
    title: "Information technology - Abstract Syntax Notation One (ASN.1): Specification of basic notation"
    author:
      org: ITU-T
    date: 2021-02
    seriesinfo:
      "ITU-T": "Recommendation X.680"
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
  RFC2986:
  RFC3161:
  RFC4211:
  RFC5126:
  RFC9321:
  EN319102-1:
    title: "Electronic Signatures and Trust Infrastructures (ESI); Procedures for Creation and Validation of AdES Digital Signatures; Part 1: Creation and Validation"
    author:
      org: ETSI
    date: 2024-06
    seriesinfo:
      "ETSI": "EN 319 102-1 v1.4.1"

--- abstract

This document defines the signedDocumentBinding certificate extension, which binds a certificate to the signed content of a single digital signature. Each certificate is created at the time of signing; the associated signing key is generated for the sole purpose of producing that one signature, is used for no other purpose, and is destroyed immediately afterwards. Certificates carrying this extension are intended to be issued without a revocation mechanism and with no expiration, which simplifies long-term validation.

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

This document takes this one step further and allows validation at any time in the future as long as trust in the Certification Authority (CA) certificate can be established.

## Design Rationale

An alternate approach for a single signature credential could have certified the hash value of the to-be-signed document. This approach was not taken because verification would be different than other digitally signed documents.  The approach taken allows existing certificate path validation and digital signature validation implementations to process certificates that are associated with one and only one document signature.

## Basic features

One signature certificates have the following common characteristics:

- They are bound to a specific document content;
- They assert that the corresponding private key was destroyed immediately after signing; and
- They are typically issued without a revocation mechanism and with no expiration.

The signedDocumentBinding extension binds the public key in the certificate to verification of the signature for the single identified document. When this extension is present, it is RECOMMENDED that the certificate not expire (notAfter is set to the GeneralizedTime value of 99991231235959Z) and the noRevAvail certificate extension [RFC9608] is also present to indicate that no revocation information is available for this certificate.

### Revocation

Traditional certificates that are re-used over time have many legitimate reasons for revocation, such as if the private key is lost or compromised. This can lead to large volumes of revocation data.

The fact that the same key is used many times exposes the key for the risks of loss, unauthorized usage, or theft. When many objects are signed with the same private key, the risk of exposure and the number of affected signed documents upon revocation increases, unless properly timestamped and properly verified.

When a signing key is used only within a single signing event, that risk of exposure is greatly reduced, and it has been shown that most usages of dedicated private keys and certificates no longer require revocation.

When certificates are used to validate digital signatures the reasons for revocation after the time of signing are not relevant when the key is used only within a single signing event. Under established signature validation procedures {{EN319102-1}}, a signature is assessed relative to a best-signature-time: the earliest time at which the signature is proven to have existed. A signing certificate that is revoked or that expires after the best-signature-time does not invalidate the signature; the past signature validation process treats such a signature as valid. The same principle underlies the grace period concept described for Cryptographic Message Syntax (CMS) Advanced Electronic Signatures (CAdES) {{RFC5126}}. Consequently, revocation information whose effective time is after the time of signing is not relevant to the validity of the signature.

Establishing the best-signature-time normally requires a trusted time source such as a time-stamp {{RFC3161}} from a trusted authority. A one signature certificate is created at the time of signing, so its issuance time establishes the time of signing directly, with the issuing CA fulfilling a role analogous to that of a time-stamp authority. Any revocation of the certificate would necessarily take effect after this time and would therefore not affect the validity of the already-created signature.

When revocation is due to key compromise, signature verifiers take extra caution because the key compromise could have taken place before the document signature was created. However, for a one signature certificate, the signing key is generated as part of the signing event and destroyed immediately afterwards, so the window during which the private key exists and could be compromised is extremely small.

### CA certificate validity

Even if the end entity certificate has infinite validity, the capability to validate the certificate is limited to the capability to trust the CA. For initial validation, in a typical scenario, this is limited by the validity period of the CA certificate.

However, a validation service may have several options available for how to handle CA trust, in particular when re-validating archived documents:

- The verifier may list the CA key and identity as trusted and treat it as a trust anchor.
- The verifier may cross certify the CA and make it available for validation to a local trusted Trust Anchor.

In addition, when a certificate repository is available, renewal of the CA certificate can preserve the ability to validate the infinite validity end entity certificate.

This specification assumes that initial validation of a signed document is performed within the validity period of the CA certificate. Realizing the full benefit of a non-expiring end entity certificate for later re-validation MAY require additional trust provisioning by the verifier.

Mechanisms for establishing trust in a CA beyond their certificate validity period are outside the scope of this specification.

### Key usage

Since the certificate includes the public key to validate a single signature, issuers MUST assert a key usage relevant to signing — digitalSignature (0), nonRepudiation (1), or both — and no other (see {{cert-content}}). Because the signedDocumentBinding extension asserts that the private key has been destroyed, the inclusion of any additional key usage value has no practical effect in any case.

## Supporting certificate issuance {#issuance}

A one signature certificate is created at the time of signing, so the signing key necessarily exists before its certificate does. Producing the verifiable signature therefore depends on first obtaining the certificate, and obtaining the certificate may itself require the key to be used as part of a certificate enrollment process. For example, the key may sign a proof of possession in a PKCS#10 certification request {{RFC2986}}, or possession may be attested without a separate signing operation using the Certificate Request Message Format {{RFC4211}}. Such issuance-supporting signatures are part of, not additional to, the single purpose for which the private key is generated: producing one verifiable signature over the bound document. The key is never used for any other purpose.

Signing a proof of possession does not expand the set of signatures that can legitimately be claimed to be valid under the certificate. The signedDocumentBinding extension is a statement by the CA, recorded in the certificate, of the one document this certificate was issued to sign; it stands as evidence of the legitimate scope of the certificate. Allowing the key to support its own certification is therefore consistent with the single-use guarantee of this specification.

This document does not mandate any particular certificate request or proof-of-possession protocol. In a typical deployment the signature service generates the key and acts as a local registration authority on behalf of the issuing CA.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Certificate content {#cert-content}

This document defines the signedDocumentBinding extension, which a CA includes in a certificate to bind that certificate to a specific signed content. The presence of this extension is the signal by which a relying party recognizes a certificate as a one signature certificate.

The following requirements apply to a certificate that includes the signedDocumentBinding extension.

Such a certificate SHOULD indicate that it has no well-defined expiration date by setting the notAfter field to the GeneralizedTime value 99991231235959Z, as defined in {{RFC5280}}.

Such a certificate SHOULD include the id-ce-noRevAvail extension in compliance with {{RFC9608}}, indicating that the certificate is not supported by any revocation mechanism.

A verifier MUST determine whether these properties apply by inspecting the notAfter field and the presence of the id-ce-noRevAvail extension.

Such a certificate MUST assert either the key usage digitalSignature (0) or nonRepudiation (1) (also known as contentCommitment) or both. The certificate MUST NOT assert any other key usage.

## The signedDocumentBinding extension

The signedDocumentBinding extension binds a certificate to a specific signed content. When present, conforming CAs SHOULD mark this extension as non-critical.


    name           id-pe-signedDocumentBinding
    OID            { id-pe 37 }
    syntax         SignedDocumentBinding
    criticality    SHOULD be FALSE

    SignedDocumentBinding ::= SEQUENCE {
    dataTbsHash     OCTET STRING,
    hashAlg         DigestAlgorithmIdentifier,
    bindingType     OBJECT IDENTIFIER OPTIONAL }


The `dataTbsHash` field MUST contain a hash of the data to be signed.

The `hashAlg` field MUST contain the AlgorithmIdentifier of the hash algorithm used to generate the `dataTbsHash` value.

The `bindingType` field MAY contain an identifier that specifies how the data to be signed is derived from the digital object to be signed.

Adding this extension to a certificate is a statement by the CA that the signing key is generated exclusively for the purpose of producing the single signature on the document bound by this extension. Serving this purpose includes performing any signing operation required to obtain this certificate, such as signing a proof of possession in a certificate request (see {{issuance}}); the key is used for no other purpose and is destroyed after the document is signed. The details for this procedure and how the destruction of the signing key is assured SHOULD be outlined in the certificate policy {{RFC3647}} of the issued certificate.

## Defined bindingType identifiers

The `bindingType` field defines how the data to be signed (`dataTbsHash`) is derived from the signed document.
This field identifies a deterministic procedure for selecting the portion of the signed content that is included in the hash computation.
When the field is omitted, the rules for the default binding type apply.

The purpose of the `dataTbsHash` value is to bind the certificate to the document being signed in order to prevent re-use of the signing key for multiple signed documents. This enforces the contract that the certificate is bound to only the single signature for which it was issued, so that no signature other than that one can legitimately be claimed to be valid under the certificate. Validators SHOULD verify that the signed document matches the certificate’s binding information.
This verification is not required for the signature to validate successfully but provides an additional safeguard against misuse or substitution of certificates.

This document defines a set of `bindingType` identifiers. Additional `bindingType` identifiers MAY be defined by future specifications.

### Default Binding

When the `bindingType` is absent, the default binding applies.
In this case, the `dataTbsHash` value is the hash of the exact data that is signed by the signature format in use.

Examples include:

- For Extensible Markup Language (XML) Signatures {{XMLDSIG11}}, the hash of the SignedInfo element.
- For CMS Signatures {{RFC5652}}, the Distinguished Encoding Rules (DER)-encoded SignedAttributes structure.
- For other formats, the data structure input directly to the signature algorithm.

This `bindingType` MUST NOT be used when the data to be signed includes either the signer certificate itself or a hash of the signer certificate. This includes JSON Web Signature (JWS) and CBOR Object Signing and Encryption (COSE) signed documents that can include signer certificates in the protected header. JWS signatures {{RFC7515}} MUST use the id-bt-jws `bindingType` and COSE signatures {{RFC8152}} MUST use the id-bt-cose binding type.

### CAdES Binding

Identifier: id-bt-cades

For CMS {{RFC5652}} or European Telecommunications Standards Institute (ETSI) CAdES {{CADES}} signatures incorporating SigningCertificate or SigningCertificateV2 attributes {{RFC5035}} in signedAttrs,
the `dataTbsHash` value is computed over the DER encoding of SignerInfo excluding any instances of SigningCertificate or SigningCertificateV2 attributes from the SignedAttributes set.

This `bindingType` also applies to Portable Document Format (PDF) {{ISOPDF2}} and ETSI PDF Advanced Electronic Signatures (PAdES) {{PADES}} signed documents when applicable due to its use of CMS for signing.

### XAdES Binding

Identifier: id-bt-xades

For ETSI XML Advanced Electronic Signatures {{XADES}}, the `dataTbsHash` value is computed over the canonicalized SignedInfo element,
with any Reference elements whose Type attribute equals "http://uri.etsi.org/01903#SignedProperties" removed prior to hashing.
This ensures that the SignedProperties element, which may contain references to the signing certificate, does not create a circular dependency. Extraction of the Reference element MUST be done by removing only the characters from the leading &lt;Reference&gt; tag up to and including the ending &lt;/Reference&gt; tag, preserving all other bytes of SignedInfo unchanged, including any white space or line feeds.

Note: This operation is purely textual and does not require XML parsing beyond locating the tag boundaries.

### JWS Binding

Identifier: id-bt-jws

For JWS {{RFC7515}}, the `dataTbsHash` value is computed over the payload only.
The protected header and any unprotected header parameters MUST NOT be included in the hash calculation.

This exclusion avoids circular dependencies where certificate data may appear in the protected header.

### COSE Binding

Identifier: id-bt-cose

For COSE signatures {{RFC8152}}, the `dataTbsHash` value is computed over the payload only.
The protected header and any unprotected header parameters MUST NOT be included in the hash calculation.

This exclusion avoids circular dependencies where certificate data may appear in the protected header.

# ASN.1 Module

This section includes the ASN.1 module {{X680}} for the signedDocumentBinding certificate extension. This module imports items from {{RFC5912}} and {{RFC6268}}.

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
         bindingType     OBJECT IDENTIFIER OPTIONAL }

       -- signedDocumentBinding Certificate Extension OID

       id-pe-signedDocumentBinding OBJECT IDENTIFIER ::= { id-pe 37 }

       -- bindingType identifiers

       id-bt OBJECT IDENTIFIER ::= {id-pkix TBD}
       id-bt-cades OBJECT IDENTIFIER ::= {id-bt 1}
       id-bt-xades OBJECT IDENTIFIER ::= {id-bt 2}
       id-bt-jws OBJECT IDENTIFIER ::= {id-bt 3}
       id-bt-cose OBJECT IDENTIFIER ::= {id-bt 4}

       END
     <CODE ENDS>

# Security Considerations

## Certificates Without Revocation

One signature certificates are intended for use with the id-ce-noRevAvail extension and no revocation mechanism. Such certificates attest only to the state of trust and correctness of procedures at the time of issuance. A verifier cannot infer this property from the signedDocumentBinding extension alone and determines it by inspecting the certificate for the id-ce-noRevAvail extension.

The Security considerations in {{RFC9608}} also applies to this document.

## Signed Document Binding

The signedDocumentBinding extension binds the certificate to specific signed content by including a hash of the data to be signed. Verification of this binding is not required for successful cryptographic validation of the signature. A signature can therefore validate correctly even if the binding is not checked.

However, a relying party SHOULD verify that the signed content matches the dataTbsHash value in the signedDocumentBinding extension. Performing this check ensures that the certificate is used only with the content for which it was issued and enforces the intended scope of the certificate.

The security model of this specification states that the associated private key is generated for the sole purpose of producing the single signature bound by the certificate, including only the signing required to obtain the certificate itself, and is then destroyed. This property holds independently of whether the binding is verified by the relying party. Nevertheless, failure to verify the binding weakens the protections provided by this specification and increases the risk of certificate substitution or unintended certificate reuse.

When verified, the signedDocumentBinding extension provides an additional safeguard against the use of the certificate for any signature other than the one for which it was issued.

# IANA Considerations {#iana}

IANA has registered id-pe-signedDocumentBinding (id-pe 37) in the SMI Security for PKIX Certificate Extension registry.

IANA is requested to assign an OBJECT IDENTIFIER for id-bt from the SMI Security for PKIX arc (id-pkix), for use as the arc for the signed document binding type identifiers defined in this document.

IANA is requested to assign an OBJECT IDENTIFIER for the ASN.1 module defined in this document.

--- back

# Example Certificates

This appendix contains non-normative example certificates that conform to
this specification.

Both certificates are issued to the subject "John Doe" by the issuing CA
"Example Org CA". Each certificate has no well-defined expiration date
(the notAfter field is set to the GeneralizedTime value 99991231235959Z),
includes the id-ce-noRevAvail extension, asserts the nonRepudiation key
usage as a critical extension, includes an authorityKeyIdentifier
extension, and includes the signedDocumentBinding extension. In both
examples the data to be signed is hashed with SHA-256, and the dataTbsHash
value is the 32-octet sequence 0x01 0x02 ... 0x20.

Note: The id-bt arc has not yet been assigned by IANA. These examples
anticipate the assignment of 1.3.6.1.5.5.7.26 to id-bt; the CAdES binding
example below therefore encodes id-bt-cades as 1.3.6.1.5.5.7.26.1
({ id-bt 1 }). Should IANA assign a different value to the id-bt arc
(see {{iana}}), these examples will be regenerated to align with it.

## Certificate with Default Binding

In this certificate the bindingType field is absent, so the default binding
applies. The signedDocumentBinding extension value contains only the
dataTbsHash and hashAlg fields:

    SignedDocumentBinding ::= SEQUENCE {
      dataTbsHash  0102030405060708090A0B0C0D0E0F10
                   1112131415161718191A1B1C1D1E1F20,
      hashAlg      id-sha256 }

    -----BEGIN CERTIFICATE-----
    MIICKTCCAc6gAwIBAgIQZe7zqov/Aufh6nVt/mAMWDAKBggqhkjOPQQDAjA8MQsw
    CQYDVQQGEwJTRTEUMBIGA1UECgwLRXhhbXBsZSBPcmcxFzAVBgNVBAMMDkV4YW1w
    bGUgT3JnIENBMCAXDTI2MTAyMDIxMjYzNFoYDzk5OTkxMjMxMjM1OTU5WjA2MQsw
    CQYDVQQGEwJTRTEUMBIGA1UECgwLRXhhbXBsZSBPcmcxETAPBgNVBAMMCEpvaG4g
    RG9lMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE8gQ14JPK+/m8VWhRh2km/mDV
    dh0Ha49htyqJLnPeb4pfBBUxR0paZbx10pQxOB/KIy/kXMzrcQVR/zsEIoTSAaOB
    tTCBsjArBgNVHSMEJDAigCDH/dB8wyv8KAJ6APwI1PziC3th/taTu8TAbFiGavtE
    wjApBgNVHQ4EIgQg6Uhmpf7aqymQdRmci2oJv1iDwcUd+jSq6ocVAQ9vkDQwDgYD
    VR0PAQH/BAQDAgZAMAkGA1UdOAQCBQAwPQYIKwYBBQUHASUEMTAvBCABAgMEBQYH
    CAkKCwwNDg8QERITFBUWFxgZGhscHR4fIDALBglghkgBZQMEAgEwCgYIKoZIzj0E
    AwIDSQAwRgIhANSdZFRgNOKSJgvOyoKBmOmdOlJ/9DyNlhOzS5SMt5bdAiEAq824
    Oyp7hCD8EGfyXsDBsP5HO4hOGvNBN+h10cr3NhY=
    -----END CERTIFICATE-----

## Certificate with CAdES Binding

In this certificate the bindingType field is set to id-bt-cades, indicating the
CAdES binding defined in this document. The signedDocumentBinding extension
value contains the dataTbsHash, hashAlg, and bindingType fields:

    SignedDocumentBinding ::= SEQUENCE {
      dataTbsHash  0102030405060708090A0B0C0D0E0F10
                   1112131415161718191A1B1C1D1E1F20,
      hashAlg      id-sha256,
      bindingType  id-bt-cades }

    -----BEGIN CERTIFICATE-----
    MIICNDCCAdmgAwIBAgIRAOPipqpq0R50YTaa4sEqrfYwCgYIKoZIzj0EAwIwPDEL
    MAkGA1UEBhMCU0UxFDASBgNVBAoMC0V4YW1wbGUgT3JnMRcwFQYDVQQDDA5FeGFt
    cGxlIE9yZyBDQTAgFw0yNjEwMjAyMTI2MzRaGA85OTk5MTIzMTIzNTk1OVowNjEL
    MAkGA1UEBhMCU0UxFDASBgNVBAoMC0V4YW1wbGUgT3JnMREwDwYDVQQDDAhKb2hu
    IERvZTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABPIENeCTyvv5vFVoUYdpJv5g
    1XYdB2uPYbcqiS5z3m+KXwQVMUdKWmW8ddKUMTgfyiMv5FzM63EFUf87BCKE0gGj
    gb8wgbwwKwYDVR0jBCQwIoAgx/3QfMMr/CgCegD8CNT84gt7Yf7Wk7vEwGxYhmr7
    RMIwKQYDVR0OBCIEIOlIZqX+2qspkHUZnItqCb9Yg8HFHfo0quqHFQEPb5A0MA4G
    A1UdDwEB/wQEAwIGQDAJBgNVHTgEAgUAMEcGCCsGAQUFBwElBDswOQQgAQIDBAUG
    BwgJCgsMDQ4PEBESExQVFhcYGRobHB0eHyAwCwYJYIZIAWUDBAIBBggrBgEFBQca
    ATAKBggqhkjOPQQDAgNJADBGAiEAs8QmUcxJcUicrcg6SCNDuGbLyOBRzMsL6OHc
    9VxPA+MCIQDvws84x3n6FP1uGIufCtZEFt5e2WDm3/3xmyMAag6/6w==
    -----END CERTIFICATE-----

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
