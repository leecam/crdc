---
title: Conditionally Released Digital Credentials
abbrev: CRDC
docname: draft-campbell-crdc-latest
category: std
ipr: trust200902
submissiontype: IETF
area: Security
workgroup: OAuth Working Group
keyword:
  - Digital Credentials
  - SD-JWT
  - mdoc
  - OpenID4VP
  - Conditional Release
  - Privacy
  - Unlinkability

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
  - ins: L. Campbell
    name: Lee Campbell
    organization: Google
    email: leecam@google.com

normative:
  RFC2119:
  RFC4086:
  RFC7515:
  RFC7516:
  RFC7517:
  RFC7518:
  RFC8174:
  RFC9901:

informative:
  RFC6750:
  RFC8414:
  RFC8705:
  RFC9180:
  RFC9458:
  ISO.18013-5:
    title: "Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL) application"
    author:
      - org: ISO/IEC
    date: 2021-09
    seriesinfo:
      ISO/IEC: 18013-5:2021
  OpenID4VCI:
    title: "OpenID for Verifiable Credential Issuance"
    author:
      - ins: T. Lodderstedt
      - ins: K. Yasuda
      - ins: T. Looker
    date: 2025
    target: https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html
  OpenID4VP:
    title: "OpenID for Verifiable Presentations"
    author:
      - ins: O. Terbu
      - ins: T. Lodderstedt
      - ins: K. Yasuda
      - ins: T. Looker
    date: 2025
    target: https://openid.net/specs/openid-4-verifiable-presentations-1_0.html
  SD-JWT-VC:
    title: "SD-JWT-based Verifiable Credentials (SD-JWT VC)"
    author:
      - ins: O. Terbu
      - ins: D. Fett
      - ins: B. Campbell
    date: 2025
    target: https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/
  W3C.DC-API:
    title: "Digital Credentials"
    author:
      - ins: M. Caceres
      - ins: K. Yasuda
      - ins: S. Goto
    date: 2025
    target: https://wicg.github.io/digital-credentials/

--- abstract

Digital Credentials (DCs), such as Selective Disclosure for JSON Web Tokens (SD-JWTs) and ISO/IEC 18013-5 mobile documents (mdocs), are issued by Issuers to Credential Managers (wallets) held by Holders. Verifiers can then request these credentials from the Credential Manager using presentation protocols such as OpenID for Verifiable Presentations (OpenID4VP) and browser APIs such as the W3C Digital Credentials API. A foundational privacy property of this three-party model is Issuer unlinkability: the Issuer does not learn when, where, or to which Verifier a given credential is presented. However, in many commercial and regulatory ecosystems, Issuers require a mechanism to charge or authorize Verifiers for credential presentations.

This specification defines a credential-format-agnostic mechanism for Conditionally Released Digital Credentials (CRDCs). During presentation, the Credential Manager encrypts a standard Digital Credential presentation using a one-time ephemeral Content Encryption Key (CEK) to produce a Releasable Credential, and encrypts the CEK under an Issuer Release Public Key alongside Issuer routing metadata to produce a Release Token. The Verifier presents the Release Token to the Issuer's Release Endpoint to authorize (and optionally bill for) the presentation and obtain the decrypted CEK, which the Verifier then uses to decrypt and validate the underlying Digital Credential—all while strictly preserving the privacy property that the Issuer never learns which Holder is presenting the credential.

--- middle

# Introduction {#introduction}

In the modern digital identity ecosystem, Digital Credentials (DCs)—such as Selective Disclosure for JSON Web Tokens (SD-JWTs) {{RFC9901}} and ISO/IEC 18013-5 mobile documents (mdocs) {{ISO.18013-5}}—are issued by Issuers to Credential Managers (commonly referred to as digital wallets) controlled by Holders. Verifiers can then request these Digital Credentials from the Holder's Credential Manager using presentation protocols such as OpenID for Verifiable Presentations {{OpenID4VP}} invoked over platform and browser surfaces like the W3C Digital Credentials API {{W3C.DC-API}}.

A core privacy tenet of this three-party model (Issuer, Holder/Credential Manager, and Verifier) is that the issuance of a credential is decoupled from its presentation. In particular, the Issuer does not participate in the presentation flow in a way that reveals the identity of the Holder to the Issuer at the time of presentation. Consequently, the Issuer does not learn which Holder is presenting their credential, nor where a specific Holder presents their credential.

At the same time, certain high-value credential ecosystems require sustainable business models where Issuers charge Verifiers for credential presentations (for example, identity verification providers, financial institutions, or authoritative registries charging relying parties per verified transaction). In traditional federated identity protocols where the Identity Provider (IdP) is directly in the loop during authentication, billing a Relying Party is straightforward but comes at the cost of the IdP learning every Relying Party a specific user visits. Conversely, in the offline or decoupled three-party Digital Credentials model, because the Credential Manager releases the credential directly to the Verifier, the Issuer has no native mechanism to enforce that a Verifier is authorized or billed for a presentation without re-introducing a Holder-tracking correlation vector.

This specification bridges this gap by defining **Conditionally Released Digital Credentials (CRDCs)**. It specifies a credential-format-agnostic cryptographic wrapper around existing credential formats (including SD-JWTs and mdocs) that allows a Credential Manager to release a credential presentation to a Verifier in an encrypted form. During presentation, the Credential Manager encrypts the underlying Digital Credential presentation using a freshly generated, one-time ephemeral symmetric Content Encryption Key (CEK)—producing the **Releasable Credential**—and encrypts that CEK under the Issuer's **Issuer Release Public Key** inside a **Release Token** that includes the Issuer's identifier. To unlock and verify the underlying credential claims, the Verifier sends the Release Token to the Issuer's Release Endpoint. Because the Release Token contains only a freshly generated random key encrypted under a public key shared across a large cohort of credentials, the Issuer can authenticate, authorize, and bill the Verifier before returning the decrypted CEK, without ever learning which Holder or credential was involved in the presentation.

## Design Rationale and Goals {#design-rationale}

This specification is built around four primary design principles:

1. **Compatibility with Existing Presentation Ecosystems**: Digital Credentials are issued to Credential Managers (wallets) by Issuers and requested by Verifiers via the W3C Digital Credentials API {{W3C.DC-API}} and presentation protocols such as OpenID4VP {{OpenID4VP}}. The conditional release mechanism integrates seamlessly into these existing flows.
2. **Preservation of Three-Party Privacy (Holder Unlinkability at the Issuer)**: In the three-party model, the Issuer does not learn where a given Holder presents their credential. Maintaining this privacy property is a strict requirement of this specification: even when the Issuer authorizes and bills the Verifier for releasing the ephemeral decryption key, the Release Token contains zero Holder-identifying or credential-instance-identifying information.
3. **Verifiable Issuer Billing and Authorization**: Issuers that wish to charge Verifiers per presentation—or enforce commercial quotas and policy checks—can require the Verifier to redeem the Release Token at the Issuer's Release Endpoint to obtain the ephemeral decryption key.
4. **Credential-Format Agnosticism**: Rather than modifying underlying credential formats or their selective disclosure and key-binding mechanisms, this specification acts as an outer cryptographic wrapper around any existing credential presentation format, such as SD-JWT {{RFC9901}} or ISO mdoc {{ISO.18013-5}}, exposed via a `-cr` format suffix convention (e.g., `dc+sd-jwt-cr`, `mso_mdoc-cr`).

## Feature Summary {#feature-summary}

This specification defines the following core components:

1. **Issuer Release Key Provisioning**: Extensions to Credential Issuer Metadata (such as in OpenID for Verifiable Credential Issuance {{OpenID4VCI}}) allowing an Issuer to publish its **Issuer Release Public Key** (`credential_release_encryption_jwks`) and **Credential Release Endpoint** (`credential_release_endpoint`).
2. **Conditionally Released Digital Credential (CRDC) Data Format**: A composite JSON structure returned as the Digital Credential presentation response, consisting of:
   * **Releasable Credential (`releasable_credential`)**: The standard underlying Digital Credential presentation (e.g., an SD-JWT+KB or an ISO mdoc `DeviceResponse`) encrypted with a one-time ephemeral AES Content Encryption Key (CEK).
   * **Release Token (`release_token`)**: A structure containing the ephemeral CEK encrypted under the Issuer Release Public Key, accompanied by unencrypted routing metadata (such as the `iss` identifier and key identifier `kid`) enabling the Verifier to locate the Issuer and redeem the key.
3. **Presentation Format Naming Convention (`-cr`)**: A standardized naming convention appending `-cr` to existing credential format identifiers (e.g., `sd-jwt-cr`, `dc+sd-jwt-cr`, `mso_mdoc-cr`) for use in presentation protocols such as OpenID4VP.
4. **Credential Release Redemption Protocol**: An HTTPS request/response protocol between the Verifier and the Issuer's Release Endpoint whereby the Verifier submits the Release Token, the Issuer authenticates and bills the Verifier, and the Issuer returns the decrypted ephemeral CEK so the Verifier can decrypt and validate the underlying Digital Credential.

## Conventions and Terminology {#terminology}

{::boilerplate bcp14-tagged}

Base64url:
: Denotes the URL-safe base64 encoding without padding defined in Section 2 of {{RFC7515}}.

Digital Credential (DC):
: A cryptographically verifiable set of claims issued by an Issuer to a Holder, and its corresponding presentation (including selective disclosures and holder key binding), such as an SD-JWT+KB {{RFC9901}} or an ISO mdoc `DeviceResponse` {{ISO.18013-5}}.

Issuer Release Public Key:
: An asymmetric public key provisioned by the Issuer to Credential Managers (e.g., via Issuer metadata in OpenID4VCI) used by the Credential Manager to encrypt the one-time ephemeral Content Encryption Key (CEK) during a presentation.

Ephemeral Content Encryption Key (CEK):
: A single-use, cryptographically random symmetric AES key generated by the Credential Manager for a single presentation to encrypt the underlying Digital Credential presentation.

Releasable Credential:
: The ciphertext structure resulting from encrypting a standard Digital Credential presentation with the one-time ephemeral CEK.

Release Token:
: A cryptographic structure containing the ephemeral CEK encrypted under the Issuer Release Public Key, together with the Issuer metadata (such as the Issuer URL and key identifier) required by the Verifier to locate the Issuer and request key release.

Conditionally Released Digital Credential (CRDC):
: The composite payload returned by the Credential Manager to the Verifier in response to a presentation request, packaging together the **Release Token** and the **Releasable Credential**.

Credential Release Endpoint:
: An HTTPS endpoint operated by the Issuer where a Verifier submits a Release Token to request decryption of the ephemeral CEK, subject to Issuer authorization and billing.

Issuer:
: An entity that issues Digital Credentials to a Holder's Credential Manager, publishes the Issuer Release Public Key, and operates the Credential Release Endpoint.

Holder:
: An entity (typically an end user) that controls Digital Credentials via a Credential Manager and presents them to Verifiers.

Credential Manager (Wallet):
: The user agent, application, or hardware/software component controlled by the Holder that stores Digital Credentials and constructs Conditionally Released Digital Credentials during presentation.

Verifier:
: A relying party that requests a Digital Credential presentation from a Credential Manager, redeems the Release Token at the Issuer's Credential Release Endpoint, decrypts the Releasable Credential, and validates the underlying Digital Credential.

# Architecture and Flow Diagram {#flow-diagram}

{{fig-flow}} illustrates the end-to-end lifecycle of a Conditionally Released Digital Credential across Issuance, Presentation, Release Token Redemption, and Verification.

~~~ ascii-art
+--------------------+      +--------------------+      +--------------------+
|                    |      | Credential Manager |      |                    |
|       Issuer       |      |      (Holder)      |      |      Verifier      |
|                    |      |                    |      |                    |
+--------------------+      +--------------------+      +--------------------+
          |                            |                           |
          | 1. Issue Credential +      |                           |
          |    Issuer Metadata         |                           |
          |    (Issuer Release         |                           |
          |     Public Key, PK_I)      |                           |
          |--------------------------->|                           |
          |                            |                           |
          |                            | 2. Presentation Request   |
          |                            |    (e.g., OpenID4VP over  |
          |                            |     DC API, format *-cr)  |
          |                            |<--------------------------|
          |                            |                           |
          |                            | 3. Generate normal DC     |
          |                            |    presentation (DC_pres) |
          |                            |    & ephemeral AES key K  |
          |                            |                           |
          |                            | 4. Encrypt DC_pres with K |
          |                            |    -> Releasable Cred     |
          |                            |    Encrypt K with PK_I +  |
          |                            |    Issuer metadata        |
          |                            |    -> Release Token       |
          |                            |                           |
          |                            | 5. Return CRDC            |
          |                            |    (Release Token +       |
          |                            |     Releasable Cred)      |
          |                            |-------------------------->|
          |                            |                           |
          | 6. Inspect Release Token metadata to identify Issuer   |
          |    & send Release Token to Issuer Release Endpoint     |
          |<-------------------------------------------------------|
          |                            |                           |
          | 7. Authenticate & bill Verifier;                       |
          |    decrypt Release Token with SK_I to recover K;       |
          |    return ephemeral AES key K to Verifier              |
          |------------------------------------------------------->|
          |                            |                           |
          |                            | 8. Decrypt Releasable     |
          |                            |    Credential using K     |
          |                            |    -> recover DC_pres     |
          |                            |                           |
          |                            | 9. Validate DC_pres       |
          |                            |    as normal              |
          +                            +                           +
~~~
{: #fig-flow title="Conditionally Released Digital Credential Issuance, Presentation, and Release Flow"}

# Concepts {#concepts}

This section describes the core components and lifecycle of Conditionally Released Digital Credentials at a conceptual level, prior to the normative data formats specified in {{data-formats}}.

## Issuer Release Public Key Provisioning {#key-provisioning}

During credential issuance (for example, using OpenID for Verifiable Credential Issuance {{OpenID4VCI}}), or via discoverable Issuer metadata, the Issuer provides the Credential Manager with an asymmetric public key termed the **Issuer Release Public Key** (`PK_I`), along with the Issuer's identifier (`iss`) and optionally a Credential Release Endpoint URL.

When an Issuer marks an issued credential as requiring conditional release, the Credential Manager stores the Issuer Release Public Key and Issuer metadata alongside the credential. To preserve Holder unlinkability, the Issuer MUST provision the same Issuer Release Public Key across a sufficiently large cohort of Holders and credentials (see {{privacy-cohort}}).

## Releasable Credential and Release Token {#releasable-cred-and-token}

When a Verifier requests a credential presentation from the Credential Manager (e.g., via OpenID4VP over the W3C Digital Credentials API), the Credential Manager first constructs the standard presentation payload for the underlying credential format—including any Holder-selected disclosures and cryptographic Holder key binding (such as an SD-JWT+KB or an ISO mdoc `DeviceResponse` bound to the Verifier's `nonce` and `aud`/`client_id`).

Instead of returning this cleartext presentation directly to the Verifier, the Credential Manager performs a two-part wrapping operation:

1. **Constructing the Releasable Credential**: The Credential Manager generates a fresh, single-use ephemeral symmetric AES Content Encryption Key (`K`). It encrypts the serialized Digital Credential presentation under `K` using an authenticated encryption with associated data (AEAD) algorithm. The resulting ciphertext structure is the **Releasable Credential**.
2. **Constructing the Release Token**: The Credential Manager encrypts the ephemeral AES key `K` using the Issuer's **Issuer Release Public Key** (`PK_I`). This encrypted key is packaged into a structure called the **Release Token**, which includes plaintext metadata identifying the Issuer (`iss`), the key identifier (`kid`), and optional release endpoint or credential type metadata so the Verifier knows where and how to redeem the token.

## Conditionally Released Digital Credential (CRDC) {#crdc-concept}

The Credential Manager packages the **Release Token** and the **Releasable Credential** together into a composite envelope called the **Conditionally Released Digital Credential (CRDC)** and returns it to the Verifier as the Digital Credential response.

From the perspective of presentation protocols such as OpenID4VP, a Conditionally Released Digital Credential can be negotiated as a distinct credential format wrapper derived from the underlying format by appending the `-cr` suffix (for example, `dc+sd-jwt-cr` or `mso_mdoc-cr`).

## Credential Release Redemption and Verification {#redemption-concept}

Upon receiving a Conditionally Released Digital Credential, the Verifier cannot immediately inspect or verify the underlying credential claims because they are encrypted within the Releasable Credential under the ephemeral AES key `K`. To unlock the credential, the Verifier performs the following steps:

1. **Inspect Metadata**: The Verifier inspects the unencrypted metadata in the **Release Token** to identify the Issuer (and, via Issuer metadata discovery or an explicit claim, the Issuer's Credential Release Endpoint).
2. **Request Release from Issuer**: The Verifier sends the **Release Token** in an authenticated request to the Issuer's Credential Release Endpoint, requesting that the presentation key be released.
3. **Issuer Approval and Billing**: The Issuer authenticates the Verifier, verifies that the Verifier is authorized to receive presentations, and optionally records a billable event or deducts from the Verifier's quota. If approved, the Issuer uses its private key (`SK_I`) corresponding to the Issuer Release Public Key (`PK_I`) to decrypt the Release Token, recovering the one-time ephemeral AES key `K`, and returns `K` to the Verifier over the TLS-protected channel.
4. **Decrypt Releasable Credential**: The Verifier uses the returned AES key `K` to decrypt the **Releasable Credential**, revealing the underlying Digital Credential presentation.
5. **Standard Credential Validation**: The Verifier validates the underlying Digital Credential presentation exactly as specified by its native format (e.g., verifying the Issuer signature, Holder key binding, disclosures, validity period, and Verifier nonce/audience binding).

Crucially, during Step 2 and Step 3, the Issuer only ever sees the Verifier's identity and the **Release Token** (which wraps a random ephemeral key `K` generated by the Credential Manager at presentation time). The Issuer never sees the Releasable Credential, any credential claims, or any identifier tied to the Holder or issuance transaction.

# Data Formats and Cryptographic Wrapper {#data-formats}

## Issuer Metadata Parameters {#issuer-metadata}

Issuers supporting Conditionally Released Digital Credentials advertise their conditional release parameters in their Credential Issuer Metadata (for example, the Credential Issuer Metadata document defined in {{OpenID4VCI}} or OAuth Authorization Server Metadata {{RFC8414}}).

The following metadata parameters are defined:

`credential_release_endpoint`:
: REQUIRED. URL of the Issuer's Credential Release Endpoint where Verifiers send Release Tokens to obtain the decrypted ephemeral key. This URL MUST use the `https` scheme.

`credential_release_encryption_jwks`:
: REQUIRED (unless `credential_release_encryption_jwks_uri` is provided). A JSON Web Key Set (JWKS) {{RFC7517}} containing one or more **Issuer Release Public Keys** used by Credential Managers to encrypt ephemeral Content Encryption Keys into Release Tokens. Each JWK MUST contain a `kid` (Key ID), `kty` (Key Type), `use` set to `"enc"`, and an `alg` (Algorithm) parameter (e.g., `"ECDH-ES"`, `"ECDH-ES+A256KW"`, or `"RSA-OAEP-256"`).

`credential_release_encryption_jwks_uri`:
: OPTIONAL. An HTTPS URL referencing a JWKS document containing the Issuer's **Issuer Release Public Keys**.

`credential_release_required`:
: OPTIONAL. A boolean value inside a specific credential configuration object in Issuer metadata indicating whether presentations of this credential MUST be wrapped as a Conditionally Released Digital Credential. Defaults to `false` if omitted.

## Releasable Credential Format {#releasable-credential-format}

The **Releasable Credential** represents the underlying Digital Credential presentation encrypted under the one-time ephemeral AES key `K` (the Content Encryption Key, or CEK).

To construct the Releasable Credential:

1. The Credential Manager produces the standard Digital Credential presentation in its native format (e.g., a UTF-8 string for an SD-JWT+KB compact serialization, or raw CBOR bytes for an ISO/IEC 18013-5 `DeviceResponse`).
2. The Credential Manager generates a fresh, cryptographically random symmetric AES Content Encryption Key `K` of length 256 bits (32 octets) (or 128 bits if `A128GCM` is used) and a random 96-bit (12-octet) Initialization Vector (`IV`).
3. The Credential Manager encrypts the raw octets of the underlying Digital Credential presentation using AES-GCM (`A256GCM` REQUIRED, `A128GCM` OPTIONAL) {{RFC7518}}.
4. To cryptographically bind the Releasable Credential to the Release Token and prevent mix-and-match substitution attacks, the ASCII bytes of the compact **Release Token** (defined in {{release-token-format}}) MUST be passed as the Additional Authenticated Data (`AAD`) to the AES-GCM encryption operation.

The `releasable_credential` is represented as a JSON object with the following members:

`enc`:
: REQUIRED. The symmetric content encryption algorithm used to encrypt the underlying credential presentation. MUST be a registered JWE `"enc"` algorithm name from {{RFC7518}}, with `"A256GCM"` as the default and Mandatory-to-Implement (MTI) algorithm.

`iv`:
: REQUIRED. The base64url-encoded Initialization Vector (nonce) used for the AEAD encryption (12 octets for AES-GCM).

`ciphertext`:
: REQUIRED. The base64url-encoded AEAD ciphertext of the underlying Digital Credential presentation.

`tag`:
: REQUIRED. The base64url-encoded Authentication Tag produced by the AEAD encryption (16 octets for AES-GCM).

## Release Token Format {#release-token-format}

The **Release Token** encapsulates the one-time ephemeral AES key `K` encrypted under the **Issuer Release Public Key**, along with the Issuer metadata needed by the Verifier to locate the Issuer and redeem the token.

The Release Token is serialized as a JSON Web Encryption (JWE) Compact Serialization string {{RFC7516}} whose plaintext payload contains the ephemeral AES key `K`.

### Release Token JWE Protected Header {#release-token-header}

The JWE Protected Header of the Release Token MUST contain the following parameters:

`alg`:
: REQUIRED. The asymmetric key encryption / key agreement algorithm used to encrypt the ephemeral key `K` under the Issuer Release Public Key (e.g., `"ECDH-ES+A256KW"`, `"ECDH-ES"`, or `"RSA-OAEP-256"`), or an HPKE-based JWE algorithm {{RFC9180}}. `"ECDH-ES+A256KW"` with curve `"P-256"` is Mandatory-to-Implement (MTI).

`enc`:
: REQUIRED. The content encryption algorithm used inside the JWE to encrypt the payload containing `K` (e.g., `"A256GCM"`).

`typ`:
: REQUIRED. Media type of the Release Token. MUST be `"crdc-release+jwe"`.

`kid`:
: REQUIRED. The Key Identifier of the **Issuer Release Public Key** used to encrypt the Release Token.

`iss`:
: REQUIRED. The Issuer identifier (an HTTPS URL with no query or fragment components) identifying the Issuer that issued the underlying Digital Credential and that can decrypt this Release Token.

`cr_ep`:
: OPTIONAL. The HTTPS URL of the Issuer's Credential Release Endpoint (`credential_release_endpoint`). Including `cr_ep` allows the Verifier to contact the Release Endpoint directly, though the Verifier SHOULD validate it against discovered Issuer metadata for `iss`.

Note: When using `"alg": "ECDH-ES"` or `"ECDH-ES+A256KW"`, the JWE Protected Header also includes the standard `"epk"` (Ephemeral Public Key) parameter generated by the Credential Manager for this single presentation.

### Release Token Plaintext Payload {#release-token-payload}

The plaintext encrypted inside the Release Token JWE MUST be a UTF-8 encoded JSON object containing at least the following members:

`k`:
: REQUIRED. The base64url-encoded raw octets of the one-time ephemeral AES Content Encryption Key `K` used to encrypt the `releasable_credential`.

`enc`:
: REQUIRED. The symmetric algorithm identifier (e.g., `"A256GCM"`) with which `K` is used.

## Conditionally Released Digital Credential (CRDC) Envelope {#crdc-envelope}

The **Conditionally Released Digital Credential (CRDC)** is the top-level JSON object returned by the Credential Manager to the Verifier in the presentation response. It contains the following fields:

`format`:
: REQUIRED. The presentation format identifier indicating a conditionally released credential (e.g., `"dc+sd-jwt-cr"` or `"mso_mdoc-cr"`), or the underlying credential format identifier when the outer protocol already signals the `-cr` format.

`release_token`:
: REQUIRED. The JWE Compact Serialization string representing the Release Token (as defined in {{release-token-format}}).

`releasable_credential`:
: REQUIRED. The JSON object representing the encrypted Digital Credential presentation (as defined in {{releasable-credential-format}}).

Example CRDC structure:

~~~ json
{
  "format": "dc+sd-jwt-cr",
  "release_token": "eyJhbGciOiJFQ0RILUVTK0EyNTZLVyIsImVuYyI6IkEyNTZHQ00iLCJ0eXAiOiJjcmRjLXJlbGVhc2UranVlIiwia2lkIjoiaXNzdWVyLXJlbGVhc2Uta2V5LTIwMjYiLCJpc3MiOiJodHRwczovL2lzc3Vlci5leGFtcGxlLmNvbSJ9...",
  "releasable_credential": {
    "enc": "A256GCM",
    "iv": "3p7bfXt93p7bfXt9",
    "ciphertext": "8f3d9a7b6c5e4f3a2b1c0d9e8f7a6b5c...",
    "tag": "1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d"
  }
}
~~~

## Presentation Format Identifiers (`-cr` Suffix Convention) {#format-identifiers}

When used with presentation protocols that negotiate credential formats (such as OpenID4VP {{OpenID4VP}} and the W3C Digital Credentials API {{W3C.DC-API}}), a Conditionally Released Digital Credential format identifier is constructed by appending the suffix **`-cr`** to the underlying credential format identifier:

| Underlying Credential Format | Underlying Format Identifier | Conditionally Released Format Identifier |
| :--- | :--- | :--- |
| SD-JWT VC {{SD-JWT-VC}} | `dc+sd-jwt` (or `vc+sd-jwt`) | `dc+sd-jwt-cr` (or `vc+sd-jwt-cr`) |
| Bare SD-JWT {{RFC9901}} | `sd-jwt` | `sd-jwt-cr` |
| ISO/IEC 18013-5 mdoc {{ISO.18013-5}} | `mso_mdoc` | `mso_mdoc-cr` |
| W3C Verifiable Credentials | `jwt_vc_json` / `ldp_vc` | `jwt_vc_json-cr` / `ldp_vc-cr` |

When a Verifier includes a `-cr` format identifier in its presentation request (or indicates support for conditional release), a Credential Manager holding a credential that requires conditional release returns the CRDC JSON envelope defined in {{crdc-envelope}}.

In addition to normal `vp_format_supported` metadata for the credential format, the Verifier MAY include the following member:

`credential_release_enc_values_supported`: An OPTIONAL non-empty JSON array of strings listing the JWE enc algorithm values [RFC7518] supported by the Verifier for decrypting a releasable_credential. Defaults to ["A256GCM"] when omitted.

# Protocol Flow and Processing Rules {#protocol-flow}

## Issuance and Key Discovery {#issuance-flow}

1. **Issuer Key Generation**: The Issuer generates one or more asymmetric key pairs (`PK_I`, `SK_I`) designated as **Issuer Release Keys** and publishes the public keys (`PK_I`) in `credential_release_encryption_jwks` alongside its `credential_release_endpoint` in its Issuer Metadata.
2. **Provisioning to Credential Manager**: During credential issuance (e.g., via OpenID4VCI), the Credential Manager retrieves and caches the Issuer's `credential_release_encryption_jwks` and `iss` identifier associated with the issued Digital Credential. The Credential Manager SHOULD periodically refresh the Issuer's release public keys in accordance with HTTP cache headers, using anonymous network transport (e.g., Oblivious HTTP {{RFC9458}} or proxy) if key rotation fetches occur post-issuance.

## Presentation by the Credential Manager {#presentation-flow}

When the Credential Manager receives a presentation request from a Verifier for a credential configured for conditional release, the Credential Manager MUST perform the following steps:

1. **Generate Underlying Presentation**: Construct the native Digital Credential presentation (`DC_pres`) according to the rules of the underlying credential format and presentation protocol. Crucially, any cryptographic Holder Key Binding (e.g., the Key Binding JWT in SD-JWT+KB or `DeviceAuth` in ISO mdoc) MUST bind to the Verifier's `nonce` and `aud`/`client_id` inside the inner `DC_pres` prior to encryption.
2. **Generate Ephemeral AES Key**: Generate a fresh, cryptographically random 256-bit AES key `K` and a 96-bit random `IV` using a cryptographically secure pseudorandom number generator (CSPRNG) {{RFC4086}}. `K` MUST NOT be reused across presentations.
3. **Create Release Token**:
   * Construct the Release Token plaintext JSON object `{"k": base64url(K), "enc": "A256GCM"}`.
   * Construct the JWE Protected Header containing `"alg"`, `"enc"`, `"typ": "crdc-release+jwe"`, `"kid"`, and `"iss"` (set to the Issuer's identifier URL).
   * Encrypt the plaintext under the Issuer's **Issuer Release Public Key** (`PK_I`) to produce the JWE Compact Serialization string `release_token`.
4. **Create Releasable Credential**:
   * Encrypt the raw octets of `DC_pres` using AES-256-GCM with key `K`, initialization vector `IV`, and Additional Authenticated Data (`AAD`) set to the ASCII bytes of `release_token`.
   * Construct the `releasable_credential` JSON object containing `enc`, `iv`, `ciphertext`, and `tag`.
5. **Return CRDC**: Package `format`, `release_token`, and `releasable_credential` into the CRDC JSON object and return it to the Verifier via the presentation protocol (e.g., over the W3C Digital Credentials API).

## Release Token Redemption (Verifier to Issuer) {#redemption-flow}

When a Verifier receives a Conditionally Released Digital Credential (`CRDC`), it MUST perform the following steps to obtain the ephemeral decryption key:

1. **Parse Release Token Header**: Base64url-decode the JWE Protected Header of `release_token` and extract the `iss` claim (and optional `cr_ep` claim).
2. **Resolve Release Endpoint**: Determine the Issuer's `credential_release_endpoint` from the Verifier's pre-configured trust store or by fetching the Issuer Metadata from `iss` (e.g., `/.well-known/openid-credential-issuer`). The Verifier MUST verify that `iss` is a trusted Issuer before sending a request.
3. **Send Release Request**: Send an HTTP `POST` request to the Issuer's `credential_release_endpoint` with content type `application/json`. The Verifier MUST authenticate itself to the Issuer (for example, using an OAuth 2.0 Bearer access token {{RFC6750}}, mutual TLS {{RFC8705}}, or `private_key_jwt` client authentication). The request body MUST be a JSON object containing:
   * `release_token`: REQUIRED. The exact `release_token` string received in the CRDC.

Example Release Request:

~~~ http
POST /release HTTP/1.1
Host: issuer.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
Content-Type: application/json

{
  "release_token": "eyJhbGciOiJFQ0RILUVTK0EyNTZLVyIsImVuYyI6IkEyNTZHQ00i..."
}
~~~

4. **Issuer Processing and Approval**: Upon receiving the request at `credential_release_endpoint`, the Issuer:
   * Authenticates the Verifier and verifies that the Verifier has an active commercial/trust relationship and sufficient quota or billing authorization.
   * Looks up the private key (`SK_I`) corresponding to the `kid` in the JWE Protected Header of `release_token`.
   * Decrypts and verifies the integrity of `release_token` using `SK_I`. If decryption fails, the Issuer MUST return an HTTP `400 Bad Request` error with error code `"invalid_release_token"` and MUST NOT bill the Verifier.
   * Records the billable transaction against the Verifier's account.
   * Returns an HTTP `200 OK` response with `Content-Type: application/json` and `Cache-Control: no-store` containing the released key object (`k` and `enc`).

Example Release Response:

~~~ http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "k": "f83j2k1l0m9n8b7v6c5x4z3a2s1d0f9g8h7j6k5l4m3",
  "enc": "A256GCM"
}
~~~

## Unwrapping and Verification by the Verifier {#verification-flow}

Upon receiving the HTTP `200 OK` response from the Issuer's Credential Release Endpoint, the Verifier MUST perform the following steps:

1. **Extract Ephemeral Key**: Base64url-decode the `k` parameter from the Issuer's response to obtain the symmetric AES key `K`.
2. **Decrypt Releasable Credential**: Using key `K`, the `iv`, `ciphertext`, and `tag` from `releasable_credential`, and the ASCII bytes of `release_token` as the `AAD`, perform AES-GCM authenticated decryption. If decryption or tag verification fails, the Verifier MUST abort processing and reject the presentation.
3. **Validate Underlying Digital Credential**: Parse the decrypted plaintext as the underlying Digital Credential presentation (`DC_pres`, e.g., an SD-JWT+KB or ISO mdoc `DeviceResponse`) and perform all standard validation checks required by the underlying format and presentation protocol, including:
   * Verifying the Issuer's cryptographic signature on the credential using the Issuer's signing public key.
   * Verifying that the Issuer of the inner credential matches the `iss` in the outer `release_token` header.
   * Verifying Holder Key Binding (`kb+jwt` or `DeviceAuth`), ensuring that the `nonce` and `aud`/`client_id` match the Verifier's presentation request.
   * Verifying selectively disclosed claims and credential validity/revocation status.

# Examples {#examples}

## Issuer Metadata Example {#example-metadata}

The following non-normative example shows an Issuer publishing its `credential_release_endpoint` and `credential_release_encryption_jwks` inside its Credential Issuer Metadata:

~~~ json
{
  "credential_issuer": "https://issuer.example.com",
  "credential_endpoint": "https://issuer.example.com/credential",
  "credential_release_endpoint": "https://issuer.example.com/release",
  "credential_release_encryption_jwks": {
    "keys": [
      {
        "kty": "EC",
        "crv": "P-256",
        "use": "enc",
        "alg": "ECDH-ES+A256KW",
        "kid": "issuer-release-key-2026-q4",
        "x": "f83OJ3D2xF1Bg8vub9tLe1gHMzV76e8Tus9uPHvRVEU",
        "y": "x_FEzRu9m36HLN_tue659LNpXW6pCyStikYjKIWI5a0"
      }
    ]
  }
}
~~~

## Conditionally Released SD-JWT (`dc+sd-jwt-cr`) {#example-sd-jwt-cr}

When wrapping an SD-JWT+KB presentation (`<Issuer-signed JWT>~<Disclosure 1>~...~<KB-JWT>`), the Credential Manager encrypts the UTF-8 octets of the complete SD-JWT+KB string into `releasable_credential` and returns:

~~~ json
{
  "format": "dc+sd-jwt-cr",
  "release_token": "eyJhbGciOiJFQ0RILUVTK0EyNTZLVyIsImVuYyI6IkEyNTZHQ00iLCJ0eXAiOiJjcmRjLXJlbGVhc2UranVlIiwia2lkIjoiaXNzdWVyLXJlbGVhc2Uta2V5LTIwMjYtcTQiLCJpc3MiOiJodHRwczovL2lzc3Vlci5leGFtcGxlLmNvbSIsImVwayI6eyJrdHkiOiJFQyIsImNydiI6IlAtMjU2IiwieCI6IjI5c19HVjB4Li4uIiwieSI6IjhsMl9IUjB5Li4uIn19.dGhpcy1pcy10aGUtencrypted-cek.aXYtYnl0ZXM.Y2lwaGVydGV4dC1ieXRlcw.dGFnLWJ5dGVz",
  "releasable_credential": {
    "enc": "A256GCM",
    "iv": "MTIzNDU2Nzg5MDEy",
    "ciphertext": "V2hlbiBkZWNyeXB0ZWQsIHRoaXMgaXMgdGhlIGZ1bGwgU0QtSldUK0tCIHN0cmluZyh...",
    "tag": "YWJjZGVmZ2hpamtsbW5vcA"
  }
}
~~~

## Conditionally Released ISO mdoc (`mso_mdoc-cr`) {#example-mdoc-cr}

When wrapping an ISO/IEC 18013-5 `DeviceResponse` CBOR structure, the Credential Manager encrypts the raw CBOR octets of `DeviceResponse` into `releasable_credential` and sets `"format": "mso_mdoc-cr"`. Upon decrypting `releasable_credential.ciphertext`, the Verifier obtains the exact CBOR bytes of `DeviceResponse` and validates the `IssuerAuth` COSE_Sign1 and `DeviceAuth` structures as normal.

# Security Considerations {#security-considerations}

## Ephemeral Key Entropy and Nonce Uniqueness {#sec-entropy}

The security of the Releasable Credential relies on the unpredictability of the one-time ephemeral AES key `K`. Credential Managers MUST generate a fresh, cryptographically random key `K` and `IV` for every presentation. Reusing `K` across presentations would allow a Verifier that paid for one presentation to decrypt another presentation without Issuer approval, and would enable correlation across presentations.

## Binding Between Releasable Credential and Release Token {#sec-binding}

By passing the ASCII representation of `release_token` as Additional Authenticated Data (`AAD`) when encrypting `releasable_credential`, the two structures are cryptographically bound. An attacker cannot strip a `release_token` from one CRDC and attach it to a different `releasable_credential` without causing AEAD tag verification to fail.

## Inner Issuer Verification Against Outer Release Token Issuer {#sec-inner-issuer}

A malicious Issuer `M` could attempt to wrap a credential issued by honest Issuer `H` under `M`'s Release Public Key in order to collect fees meant for `H`. To prevent this, as specified in {{verification-flow}}, the Verifier MUST verify after decrypting the Releasable Credential that the cryptographic Issuer of the inner Digital Credential matches the `iss` claimed in the outer `release_token` JWE Protected Header.

## Inner Holder Key Binding {#sec-holder-key-binding}

The CRDC wrapper provides conditional confidentiality of the presentation until Issuer approval, but does not replace the inner credential's Holder Key Binding. The inner Digital Credential presentation (e.g., SD-JWT+KB or mdoc `DeviceResponse`) MUST still include cryptographic Holder Key Binding over the Verifier's `nonce` and `aud`/`client_id`. This ensures that even after decryption, the Verifier has cryptographic proof that the Holder consented to present the credential to that specific Verifier in that specific session.

# Privacy Considerations {#privacy-considerations}

## Unlinkability Against the Issuer (Shared Key Cohort Requirements) {#privacy-cohort}

The primary privacy guarantee of Conditionally Released Digital Credentials is that the Issuer can authorize and bill a Verifier for a presentation without learning which Holder's credential is being presented.

To uphold this guarantee:

1. **No Per-Holder Release Keys**: The Issuer MUST NOT provision unique or fine-grained **Issuer Release Public Keys** (`PK_I` / `kid`) to individual Holders or small cohorts of Holders. If an Issuer assigned a distinct `PK_I` to each Holder, the `kid` (or successful trial decryption) at the Release Endpoint would immediately reveal which Holder is presenting their credential to the Verifier. Issuers MUST share each Issuer Release Public Key across a large anonymity cohort (e.g., all credentials of a given type issued within a broad time window).
2. **No Holder-Identifying Metadata in the Release Token**: The Credential Manager MUST NOT include any Holder identifier, credential serial number, issuance timestamp, or static salt inside either the JWE Protected Header or the encrypted payload of the `release_token`. Both the ephemeral asymmetric key (`epk`) used in JWE key agreement and the ephemeral symmetric key (`K`) inside the JWE payload MUST be freshly generated at presentation time.

## Confidentiality of Claims Against the Issuer {#privacy-confidentiality}

Because the Verifier only sends the `release_token` to the Issuer's Release Endpoint and never sends the `releasable_credential`, the Issuer never sees the underlying credential, the Holder's selectively disclosed claims, or the Verifier's session nonce.

## Timing and Network Correlation Mitigations {#privacy-timing}

If an Issuer issues a credential to a Holder and the Holder immediately presents that credential to a Verifier milliseconds later, the Issuer could attempt to correlate the issuance event with the Verifier's Release Token redemption by timing alone. Credential Managers SHOULD pre-provision credentials and Issuer Release Public Keys ahead of presentation time, or introduce jitter when issuance and presentation occur back-to-back.

# IANA Considerations {#iana-considerations}

## OAuth Authorization Server / Credential Issuer Metadata Registry {#iana-metadata}

This specification requests registration of the following metadata parameters:

* `credential_release_endpoint`: URL of the Issuer's Credential Release Endpoint.
* `credential_release_encryption_jwks`: JWKS containing the Issuer Release Public Keys.
* `credential_release_encryption_jwks_uri`: URL of the JWKS containing the Issuer Release Public Keys.

## Media Type Registration {#iana-media-type}

This section registers the `"application/crdc-release+jwe"` media type (`typ` shorthand `"crdc-release+jwe"`) in the IANA "Media Types" registry to identify a Release Token JWE.

--- back
