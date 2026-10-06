| Internet-Draft | CRDC                 | October 2026 |
|----------------|----------------------|--------------|
| Campbell       | Expires 9 April 2027 | \[Page\]     |

<div id="external-metadata" class="document-information">

</div>

<div id="internal-metadata" class="document-information">

Workgroup:  
OAuth Working Group

Internet-Draft:  
draft-campbell-crdc-00

Published:  
6 October 2026

Intended Status:  
Standards Track

Expires:  
9 April 2027

Author:  
<div class="author">

<div class="author-name">

L. Campbell

</div>

<div class="org">

Google

</div>

</div>

</div>

# Conditionally Released Digital Credentials

<div id="section-abstract" class="section">

## <a href="#abstract" class="selfRef">Abstract</a>

Digital Credentials (DCs), such as Selective Disclosure for JSON Web Tokens (SD-JWTs) and ISO/IEC 18013-5 mobile documents (mdocs), are issued by Issuers to Credential Managers (wallets) held by Holders. Verifiers can then request these credentials from the Credential Manager using presentation protocols such as OpenID for Verifiable Presentations (OpenID4VP) and browser APIs such as the W3C Digital Credentials API. A foundational privacy property of this three-party model is Issuer unlinkability: the Issuer does not learn when, where, or to which Verifier a given credential is presented. However, in many commercial and regulatory ecosystems, Issuers require a mechanism to charge or authorize Verifiers for credential presentations.<a href="#section-abstract-1" class="pilcrow">¶</a>

This specification defines a credential-format-agnostic mechanism for Conditionally Released Digital Credentials (CRDCs). During presentation, the Credential Manager encrypts a standard Digital Credential presentation using a one-time ephemeral Content Encryption Key (CEK) to produce a Releasable Credential, and encrypts the CEK under an Issuer Release Public Key alongside Issuer routing metadata to produce a Release Token. The Verifier presents the Release Token to the Issuer's Release Endpoint to authorize (and optionally bill for) the presentation and obtain the decrypted CEK, which the Verifier then uses to decrypt and validate the underlying Digital Credential—all while strictly preserving the privacy property that the Issuer never learns which Holder is presenting the credential.<a href="#section-abstract-2" class="pilcrow">¶</a>

</div>

<div id="status-of-memo">

<div id="section-boilerplate.1" class="section">

## <a href="#name-status-of-this-memo" class="section-name selfRef">Status of This Memo</a>

This Internet-Draft is submitted in full conformance with the provisions of BCP 78 and BCP 79.<a href="#section-boilerplate.1-1" class="pilcrow">¶</a>

Internet-Drafts are working documents of the Internet Engineering Task Force (IETF). Note that other groups may also distribute working documents as Internet-Drafts. The list of current Internet-Drafts is at <https://datatracker.ietf.org/drafts/current/>.<a href="#section-boilerplate.1-2" class="pilcrow">¶</a>

Internet-Drafts are draft documents valid for a maximum of six months and may be updated, replaced, or obsoleted by other documents at any time. It is inappropriate to use Internet-Drafts as reference material or to cite them other than as "work in progress."<a href="#section-boilerplate.1-3" class="pilcrow">¶</a>

This Internet-Draft will expire on 9 April 2027.<a href="#section-boilerplate.1-4" class="pilcrow">¶</a>

</div>

</div>

<div id="copyright">

<div id="section-boilerplate.2" class="section">

## <a href="#name-copyright-notice" class="section-name selfRef">Copyright Notice</a>

Copyright (c) 2026 IETF Trust and the persons identified as the document authors. All rights reserved.<a href="#section-boilerplate.2-1" class="pilcrow">¶</a>

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (<https://trustee.ietf.org/license-info>) in effect on the date of publication of this document. Please review these documents carefully, as they describe your rights and restrictions with respect to this document. Code Components extracted from this document must include Revised BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Revised BSD License.<a href="#section-boilerplate.2-2" class="pilcrow">¶</a>

</div>

</div>

<div id="toc">

<div id="section-toc.1" class="section">

<a href="#" class="toplink" onclick="scroll(0,0)">▲</a>

## <a href="#name-table-of-contents" class="section-name selfRef">Table of Contents</a>

- <div id="section-toc.1-1.1">

  <a href="#section-1" class="auto internal xref">1</a>.  <a href="#name-introduction" class="internal xref">Introduction</a>

  - <div id="section-toc.1-1.1.2.1">

    <a href="#section-1.1" class="auto internal xref">1.1</a>.  <a href="#name-design-rationale-and-goals" class="internal xref">Design Rationale and Goals</a>

    </div>

  - <div id="section-toc.1-1.1.2.2">

    <a href="#section-1.2" class="auto internal xref">1.2</a>.  <a href="#name-feature-summary" class="internal xref">Feature Summary</a>

    </div>

  - <div id="section-toc.1-1.1.2.3">

    <a href="#section-1.3" class="auto internal xref">1.3</a>.  <a href="#name-conventions-and-terminology" class="internal xref">Conventions and Terminology</a>

    </div>

  </div>

- <div id="section-toc.1-1.2">

  <a href="#section-2" class="auto internal xref">2</a>.  <a href="#name-architecture-and-flow-diagr" class="internal xref">Architecture and Flow Diagram</a>

  </div>

- <div id="section-toc.1-1.3">

  <a href="#section-3" class="auto internal xref">3</a>.  <a href="#name-concepts" class="internal xref">Concepts</a>

  - <div id="section-toc.1-1.3.2.1">

    <a href="#section-3.1" class="auto internal xref">3.1</a>.  <a href="#name-issuer-release-public-key-p" class="internal xref">Issuer Release Public Key Provisioning</a>

    </div>

  - <div id="section-toc.1-1.3.2.2">

    <a href="#section-3.2" class="auto internal xref">3.2</a>.  <a href="#name-releasable-credential-and-r" class="internal xref">Releasable Credential and Release Token</a>

    </div>

  - <div id="section-toc.1-1.3.2.3">

    <a href="#section-3.3" class="auto internal xref">3.3</a>.  <a href="#name-conditionally-released-digit" class="internal xref">Conditionally Released Digital Credential (CRDC)</a>

    </div>

  - <div id="section-toc.1-1.3.2.4">

    <a href="#section-3.4" class="auto internal xref">3.4</a>.  <a href="#name-credential-release-redempti" class="internal xref">Credential Release Redemption and Verification</a>

    </div>

  </div>

- <div id="section-toc.1-1.4">

  <a href="#section-4" class="auto internal xref">4</a>.  <a href="#name-data-formats-and-cryptograp" class="internal xref">Data Formats and Cryptographic Wrapper</a>

  - <div id="section-toc.1-1.4.2.1">

    <a href="#section-4.1" class="auto internal xref">4.1</a>.  <a href="#name-issuer-metadata-parameters" class="internal xref">Issuer Metadata Parameters</a>

    </div>

  - <div id="section-toc.1-1.4.2.2">

    <a href="#section-4.2" class="auto internal xref">4.2</a>.  <a href="#name-releasable-credential-forma" class="internal xref">Releasable Credential Format</a>

    </div>

  - <div id="section-toc.1-1.4.2.3">

    <a href="#section-4.3" class="auto internal xref">4.3</a>.  <a href="#name-release-token-format" class="internal xref">Release Token Format</a>

    - <div id="section-toc.1-1.4.2.3.2.1">

      <a href="#section-4.3.1" class="auto internal xref">4.3.1</a>.  <a href="#name-release-token-jwe-protected" class="internal xref">Release Token JWE Protected Header</a>

      </div>

    - <div id="section-toc.1-1.4.2.3.2.2">

      <a href="#section-4.3.2" class="auto internal xref">4.3.2</a>.  <a href="#name-release-token-plaintext-pay" class="internal xref">Release Token Plaintext Payload</a>

      </div>

    </div>

  - <div id="section-toc.1-1.4.2.4">

    <a href="#section-4.4" class="auto internal xref">4.4</a>.  <a href="#name-conditionally-released-digita" class="internal xref">Conditionally Released Digital Credential (CRDC) Envelope</a>

    </div>

  - <div id="section-toc.1-1.4.2.5">

    <a href="#section-4.5" class="auto internal xref">4.5</a>.  <a href="#name-presentation-format-identif" class="internal xref">Presentation Format Identifiers (<code>-cr</code> Suffix Convention)</a>

    </div>

  </div>

- <div id="section-toc.1-1.5">

  <a href="#section-5" class="auto internal xref">5</a>.  <a href="#name-protocol-flow-and-processin" class="internal xref">Protocol Flow and Processing Rules</a>

  - <div id="section-toc.1-1.5.2.1">

    <a href="#section-5.1" class="auto internal xref">5.1</a>.  <a href="#name-issuance-and-key-discovery" class="internal xref">Issuance and Key Discovery</a>

    </div>

  - <div id="section-toc.1-1.5.2.2">

    <a href="#section-5.2" class="auto internal xref">5.2</a>.  <a href="#name-presentation-by-the-credent" class="internal xref">Presentation by the Credential Manager</a>

    </div>

  - <div id="section-toc.1-1.5.2.3">

    <a href="#section-5.3" class="auto internal xref">5.3</a>.  <a href="#name-release-token-redemption-ve" class="internal xref">Release Token Redemption (Verifier to Issuer)</a>

    </div>

  - <div id="section-toc.1-1.5.2.4">

    <a href="#section-5.4" class="auto internal xref">5.4</a>.  <a href="#name-unwrapping-and-verification" class="internal xref">Unwrapping and Verification by the Verifier</a>

    </div>

  </div>

- <div id="section-toc.1-1.6">

  <a href="#section-6" class="auto internal xref">6</a>.  <a href="#name-examples" class="internal xref">Examples</a>

  - <div id="section-toc.1-1.6.2.1">

    <a href="#section-6.1" class="auto internal xref">6.1</a>.  <a href="#name-issuer-metadata-example" class="internal xref">Issuer Metadata Example</a>

    </div>

  - <div id="section-toc.1-1.6.2.2">

    <a href="#section-6.2" class="auto internal xref">6.2</a>.  <a href="#name-conditionally-released-sd-j" class="internal xref">Conditionally Released SD-JWT (<code>dc+sd-jwt-cr</code>)</a>

    </div>

  - <div id="section-toc.1-1.6.2.3">

    <a href="#section-6.3" class="auto internal xref">6.3</a>.  <a href="#name-conditionally-released-iso-" class="internal xref">Conditionally Released ISO mdoc (<code>mso_mdoc-cr</code>)</a>

    </div>

  </div>

- <div id="section-toc.1-1.7">

  <a href="#section-7" class="auto internal xref">7</a>.  <a href="#name-security-considerations" class="internal xref">Security Considerations</a>

  - <div id="section-toc.1-1.7.2.1">

    <a href="#section-7.1" class="auto internal xref">7.1</a>.  <a href="#name-ephemeral-key-entropy-and-n" class="internal xref">Ephemeral Key Entropy and Nonce Uniqueness</a>

    </div>

  - <div id="section-toc.1-1.7.2.2">

    <a href="#section-7.2" class="auto internal xref">7.2</a>.  <a href="#name-binding-between-releasable-" class="internal xref">Binding Between Releasable Credential and Release Token</a>

    </div>

  - <div id="section-toc.1-1.7.2.3">

    <a href="#section-7.3" class="auto internal xref">7.3</a>.  <a href="#name-inner-issuer-verification-a" class="internal xref">Inner Issuer Verification Against Outer Release Token Issuer</a>

    </div>

  - <div id="section-toc.1-1.7.2.4">

    <a href="#section-7.4" class="auto internal xref">7.4</a>.  <a href="#name-inner-holder-key-binding" class="internal xref">Inner Holder Key Binding</a>

    </div>

  </div>

- <div id="section-toc.1-1.8">

  <a href="#section-8" class="auto internal xref">8</a>.  <a href="#name-privacy-considerations" class="internal xref">Privacy Considerations</a>

  - <div id="section-toc.1-1.8.2.1">

    <a href="#section-8.1" class="auto internal xref">8.1</a>.  <a href="#name-unlinkability-against-the-i" class="internal xref">Unlinkability Against the Issuer (Shared Key Cohort Requirements)</a>

    </div>

  - <div id="section-toc.1-1.8.2.2">

    <a href="#section-8.2" class="auto internal xref">8.2</a>.  <a href="#name-confidentiality-of-claims-a" class="internal xref">Confidentiality of Claims Against the Issuer</a>

    </div>

  - <div id="section-toc.1-1.8.2.3">

    <a href="#section-8.3" class="auto internal xref">8.3</a>.  <a href="#name-timing-and-network-correlat" class="internal xref">Timing and Network Correlation Mitigations</a>

    </div>

  </div>

- <div id="section-toc.1-1.9">

  <a href="#section-9" class="auto internal xref">9</a>.  <a href="#name-iana-considerations" class="internal xref">IANA Considerations</a>

  - <div id="section-toc.1-1.9.2.1">

    <a href="#section-9.1" class="auto internal xref">9.1</a>.  <a href="#name-oauth-authorization-server-" class="internal xref">OAuth Authorization Server / Credential Issuer Metadata Registry</a>

    </div>

  - <div id="section-toc.1-1.9.2.2">

    <a href="#section-9.2" class="auto internal xref">9.2</a>.  <a href="#name-media-type-registration" class="internal xref">Media Type Registration</a>

    </div>

  </div>

- <div id="section-toc.1-1.10">

  <a href="#section-10" class="auto internal xref">10</a>. <a href="#name-references" class="internal xref">References</a>

  - <div id="section-toc.1-1.10.2.1">

    <a href="#section-10.1" class="auto internal xref">10.1</a>.  <a href="#name-normative-references" class="internal xref">Normative References</a>

    </div>

  - <div id="section-toc.1-1.10.2.2">

    <a href="#section-10.2" class="auto internal xref">10.2</a>.  <a href="#name-informative-references" class="internal xref">Informative References</a>

    </div>

  </div>

- <div id="section-toc.1-1.11">

  <a href="#appendix-A" class="auto internal xref"></a><a href="#name-authors-address" class="internal xref">Author's Address</a>

  </div>

</div>

</div>

<div id="introduction">

<div id="section-1" class="section">

## <a href="#section-1" class="section-number selfRef">1.</a> <a href="#name-introduction" class="section-name selfRef">Introduction</a>

In the modern digital identity ecosystem, Digital Credentials (DCs)—such as Selective Disclosure for JSON Web Tokens (SD-JWTs) \[<a href="#RFC9901" class="cite xref">RFC9901</a>\] and ISO/IEC 18013-5 mobile documents (mdocs) \[<a href="#ISO.18013-5" class="cite xref">ISO.18013-5</a>\]—are issued by Issuers to Credential Managers (commonly referred to as digital wallets) controlled by Holders. Verifiers can then request these Digital Credentials from the Holder's Credential Manager using presentation protocols such as OpenID for Verifiable Presentations \[<a href="#OpenID4VP" class="cite xref">OpenID4VP</a>\] invoked over platform and browser surfaces like the W3C Digital Credentials API \[<a href="#W3C.DC-API" class="cite xref">W3C.DC-API</a>\].<a href="#section-1-1" class="pilcrow">¶</a>

A core privacy tenet of this three-party model (Issuer, Holder/Credential Manager, and Verifier) is that the issuance of a credential is decoupled from its presentation. In particular, the Issuer does not participate in the presentation flow in a way that reveals the identity of the Holder to the Issuer at the time of presentation. Consequently, the Issuer does not learn which Holder is presenting their credential, nor where a specific Holder presents their credential.<a href="#section-1-2" class="pilcrow">¶</a>

At the same time, certain high-value credential ecosystems require sustainable business models where Issuers charge Verifiers for credential presentations (for example, identity verification providers, financial institutions, or authoritative registries charging relying parties per verified transaction). In traditional federated identity protocols where the Identity Provider (IdP) is directly in the loop during authentication, billing a Relying Party is straightforward but comes at the cost of the IdP learning every Relying Party a specific user visits. Conversely, in the offline or decoupled three-party Digital Credentials model, because the Credential Manager releases the credential directly to the Verifier, the Issuer has no native mechanism to enforce that a Verifier is authorized or billed for a presentation without re-introducing a Holder-tracking correlation vector.<a href="#section-1-3" class="pilcrow">¶</a>

This specification bridges this gap by defining **Conditionally Released Digital Credentials (CRDCs)**. It specifies a credential-format-agnostic cryptographic wrapper around existing credential formats (including SD-JWTs and mdocs) that allows a Credential Manager to release a credential presentation to a Verifier in an encrypted form. During presentation, the Credential Manager encrypts the underlying Digital Credential presentation using a freshly generated, one-time ephemeral symmetric Content Encryption Key (CEK)—producing the **Releasable Credential**—and encrypts that CEK under the Issuer's **Issuer Release Public Key** inside a **Release Token** that includes the Issuer's identifier. To unlock and verify the underlying credential claims, the Verifier sends the Release Token to the Issuer's Release Endpoint. Because the Release Token contains only a freshly generated random key encrypted under a public key shared across a large cohort of credentials, the Issuer can authenticate, authorize, and bill the Verifier before returning the decrypted CEK, without ever learning which Holder or credential was involved in the presentation.<a href="#section-1-4" class="pilcrow">¶</a>

<div id="design-rationale">

<div id="section-1.1" class="section">

### <a href="#section-1.1" class="section-number selfRef">1.1.</a> <a href="#name-design-rationale-and-goals" class="section-name selfRef">Design Rationale and Goals</a>

This specification is built around four primary design principles:<a href="#section-1.1-1" class="pilcrow">¶</a>

1.  <div id="section-1.1-2.1">

    **Compatibility with Existing Presentation Ecosystems**: Digital Credentials are issued to Credential Managers (wallets) by Issuers and requested by Verifiers via the W3C Digital Credentials API \[<a href="#W3C.DC-API" class="cite xref">W3C.DC-API</a>\] and presentation protocols such as OpenID4VP \[<a href="#OpenID4VP" class="cite xref">OpenID4VP</a>\]. The conditional release mechanism integrates seamlessly into these existing flows.<a href="#section-1.1-2.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-1.1-2.2">

    **Preservation of Three-Party Privacy (Holder Unlinkability at the Issuer)**: In the three-party model, the Issuer does not learn where a given Holder presents their credential. Maintaining this privacy property is a strict requirement of this specification: even when the Issuer authorizes and bills the Verifier for releasing the ephemeral decryption key, the Release Token contains zero Holder-identifying or credential-instance-identifying information.<a href="#section-1.1-2.2.1" class="pilcrow">¶</a>

    </div>

3.  <div id="section-1.1-2.3">

    **Verifiable Issuer Billing and Authorization**: Issuers that wish to charge Verifiers per presentation—or enforce commercial quotas and policy checks—can require the Verifier to redeem the Release Token at the Issuer's Release Endpoint to obtain the ephemeral decryption key.<a href="#section-1.1-2.3.1" class="pilcrow">¶</a>

    </div>

4.  <div id="section-1.1-2.4">

    **Credential-Format Agnosticism**: Rather than modifying underlying credential formats or their selective disclosure and key-binding mechanisms, this specification acts as an outer cryptographic wrapper around any existing credential presentation format, such as SD-JWT \[<a href="#RFC9901" class="cite xref">RFC9901</a>\] or ISO mdoc \[<a href="#ISO.18013-5" class="cite xref">ISO.18013-5</a>\], exposed via a `-cr` format suffix convention (e.g., `dc+sd-jwt-cr`, `mso_mdoc-cr`).<a href="#section-1.1-2.4.1" class="pilcrow">¶</a>

    </div>

</div>

</div>

<div id="feature-summary">

<div id="section-1.2" class="section">

### <a href="#section-1.2" class="section-number selfRef">1.2.</a> <a href="#name-feature-summary" class="section-name selfRef">Feature Summary</a>

This specification defines the following core components:<a href="#section-1.2-1" class="pilcrow">¶</a>

1.  <div id="section-1.2-2.1">

    **Issuer Release Key Provisioning**: Extensions to Credential Issuer Metadata (such as in OpenID for Verifiable Credential Issuance \[<a href="#OpenID4VCI" class="cite xref">OpenID4VCI</a>\]) allowing an Issuer to publish its **Issuer Release Public Key** (`credential_release_encryption_jwks`) and **Credential Release Endpoint** (`credential_release_endpoint`).<a href="#section-1.2-2.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-1.2-2.2">

    **Conditionally Released Digital Credential (CRDC) Data Format**: A composite JSON structure returned as the Digital Credential presentation response, consisting of:<a href="#section-1.2-2.2.1" class="pilcrow">¶</a>

    - <div id="section-1.2-2.2.2.1">

      **Releasable Credential (`releasable_credential`)**: The standard underlying Digital Credential presentation (e.g., an SD-JWT+KB or an ISO mdoc `DeviceResponse`) encrypted with a one-time ephemeral AES Content Encryption Key (CEK).<a href="#section-1.2-2.2.2.1.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-1.2-2.2.2.2">

      **Release Token (`release_token`)**: A structure containing the ephemeral CEK encrypted under the Issuer Release Public Key, accompanied by unencrypted routing metadata (such as the `iss` identifier and key identifier `kid`) enabling the Verifier to locate the Issuer and redeem the key.<a href="#section-1.2-2.2.2.2.1" class="pilcrow">¶</a>

      </div>

    </div>

3.  <div id="section-1.2-2.3">

    **Presentation Format Naming Convention (`-cr`)**: A standardized naming convention appending `-cr` to existing credential format identifiers (e.g., `sd-jwt-cr`, `dc+sd-jwt-cr`, `mso_mdoc-cr`) for use in presentation protocols such as OpenID4VP.<a href="#section-1.2-2.3.1" class="pilcrow">¶</a>

    </div>

4.  <div id="section-1.2-2.4">

    **Credential Release Redemption Protocol**: An HTTPS request/response protocol between the Verifier and the Issuer's Release Endpoint whereby the Verifier submits the Release Token, the Issuer authenticates and bills the Verifier, and the Issuer returns the decrypted ephemeral CEK so the Verifier can decrypt and validate the underlying Digital Credential.<a href="#section-1.2-2.4.1" class="pilcrow">¶</a>

    </div>

</div>

</div>

<div id="terminology">

<div id="section-1.3" class="section">

### <a href="#section-1.3" class="section-number selfRef">1.3.</a> <a href="#name-conventions-and-terminology" class="section-name selfRef">Conventions and Terminology</a>

The key words "<span class="bcp14">MUST</span>", "<span class="bcp14">MUST NOT</span>", "<span class="bcp14">REQUIRED</span>", "<span class="bcp14">SHALL</span>", "<span class="bcp14">SHALL NOT</span>", "<span class="bcp14">SHOULD</span>", "<span class="bcp14">SHOULD NOT</span>", "<span class="bcp14">RECOMMENDED</span>", "<span class="bcp14">NOT RECOMMENDED</span>", "<span class="bcp14">MAY</span>", and "<span class="bcp14">OPTIONAL</span>" in this document are to be interpreted as described in BCP 14 \[<a href="#RFC2119" class="cite xref">RFC2119</a>\] \[<a href="#RFC8174" class="cite xref">RFC8174</a>\] when, and only when, they appear in all capitals, as shown here.<a href="#section-1.3-1" class="pilcrow">¶</a>

<span class="break"></span>

Base64url:  
Denotes the URL-safe base64 encoding without padding defined in Section 2 of \[<a href="#RFC7515" class="cite xref">RFC7515</a>\].<a href="#section-1.3-2.2.1" class="pilcrow">¶</a>

Digital Credential (DC):  
A cryptographically verifiable set of claims issued by an Issuer to a Holder, and its corresponding presentation (including selective disclosures and holder key binding), such as an SD-JWT+KB \[<a href="#RFC9901" class="cite xref">RFC9901</a>\] or an ISO mdoc `DeviceResponse` \[<a href="#ISO.18013-5" class="cite xref">ISO.18013-5</a>\].<a href="#section-1.3-2.4.1" class="pilcrow">¶</a>

Issuer Release Public Key:  
An asymmetric public key provisioned by the Issuer to Credential Managers (e.g., via Issuer metadata in OpenID4VCI) used by the Credential Manager to encrypt the one-time ephemeral Content Encryption Key (CEK) during a presentation.<a href="#section-1.3-2.6.1" class="pilcrow">¶</a>

Ephemeral Content Encryption Key (CEK):  
A single-use, cryptographically random symmetric AES key generated by the Credential Manager for a single presentation to encrypt the underlying Digital Credential presentation.<a href="#section-1.3-2.8.1" class="pilcrow">¶</a>

Releasable Credential:  
The ciphertext structure resulting from encrypting a standard Digital Credential presentation with the one-time ephemeral CEK.<a href="#section-1.3-2.10.1" class="pilcrow">¶</a>

Release Token:  
A cryptographic structure containing the ephemeral CEK encrypted under the Issuer Release Public Key, together with the Issuer metadata (such as the Issuer URL and key identifier) required by the Verifier to locate the Issuer and request key release.<a href="#section-1.3-2.12.1" class="pilcrow">¶</a>

Conditionally Released Digital Credential (CRDC):  
The composite payload returned by the Credential Manager to the Verifier in response to a presentation request, packaging together the **Release Token** and the **Releasable Credential**.<a href="#section-1.3-2.14.1" class="pilcrow">¶</a>

Credential Release Endpoint:  
An HTTPS endpoint operated by the Issuer where a Verifier submits a Release Token to request decryption of the ephemeral CEK, subject to Issuer authorization and billing.<a href="#section-1.3-2.16.1" class="pilcrow">¶</a>

Issuer:  
An entity that issues Digital Credentials to a Holder's Credential Manager, publishes the Issuer Release Public Key, and operates the Credential Release Endpoint.<a href="#section-1.3-2.18.1" class="pilcrow">¶</a>

Holder:  
An entity (typically an end user) that controls Digital Credentials via a Credential Manager and presents them to Verifiers.<a href="#section-1.3-2.20.1" class="pilcrow">¶</a>

Credential Manager (Wallet):  
The user agent, application, or hardware/software component controlled by the Holder that stores Digital Credentials and constructs Conditionally Released Digital Credentials during presentation.<a href="#section-1.3-2.22.1" class="pilcrow">¶</a>

Verifier:  
A relying party that requests a Digital Credential presentation from a Credential Manager, redeems the Release Token at the Issuer's Credential Release Endpoint, decrypts the Releasable Credential, and validates the underlying Digital Credential.<a href="#section-1.3-2.24.1" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="flow-diagram">

<div id="section-2" class="section">

## <a href="#section-2" class="section-number selfRef">2.</a> <a href="#name-architecture-and-flow-diagr" class="section-name selfRef">Architecture and Flow Diagram</a>

<a href="#fig-flow" class="auto internal xref">Figure 1</a> illustrates the end-to-end lifecycle of a Conditionally Released Digital Credential across Issuance, Presentation, Release Token Redemption, and Verification.<a href="#section-2-1" class="pilcrow">¶</a>

<span id="name-conditionally-released-digi"></span>

<div id="fig-flow">

<figure id="figure-1">
<div id="section-2-2.1" class="alignLeft art-ascii-art art-text artwork">
<pre><code>+--------------------+      +--------------------+      +--------------------+
|                    |      | Credential Manager |      |                    |
|       Issuer       |      |      (Holder)      |      |      Verifier      |
|                    |      |                    |      |                    |
+--------------------+      +--------------------+      +--------------------+
          |                            |                           |
          | 1. Issue Credential +      |                           |
          |    Issuer Metadata         |                           |
          |    (Issuer Release         |                           |
          |     Public Key, PK_I)      |                           |
          |---------------------------&gt;|                           |
          |                            |                           |
          |                            | 2. Presentation Request   |
          |                            |    (e.g., OpenID4VP over  |
          |                            |     DC API, format *-cr)  |
          |                            |&lt;--------------------------|
          |                            |                           |
          |                            | 3. Generate normal DC     |
          |                            |    presentation (DC_pres) |
          |                            |    &amp; ephemeral AES key K  |
          |                            |                           |
          |                            | 4. Encrypt DC_pres with K |
          |                            |    -&gt; Releasable Cred     |
          |                            |    Encrypt K with PK_I +  |
          |                            |    Issuer metadata        |
          |                            |    -&gt; Release Token       |
          |                            |                           |
          |                            | 5. Return CRDC            |
          |                            |    (Release Token +       |
          |                            |     Releasable Cred)      |
          |                            |--------------------------&gt;|
          |                            |                           |
          | 6. Inspect Release Token metadata to identify Issuer   |
          |    &amp; send Release Token to Issuer Release Endpoint     |
          |&lt;-------------------------------------------------------|
          |                            |                           |
          | 7. Authenticate &amp; bill Verifier;                       |
          |    decrypt Release Token with SK_I to recover K;       |
          |    return ephemeral AES key K to Verifier              |
          |-------------------------------------------------------&gt;|
          |                            |                           |
          |                            | 8. Decrypt Releasable     |
          |                            |    Credential using K     |
          |                            |    -&gt; recover DC_pres     |
          |                            |                           |
          |                            | 9. Validate DC_pres       |
          |                            |    as normal              |
          +                            +                           +</code></pre>
</div>
<figcaption><a href="#figure-1" class="selfRef">Figure 1</a>: <a href="#name-conditionally-released-digi" class="selfRef">Conditionally Released Digital Credential Issuance, Presentation, and Release Flow</a></figcaption>
</figure>

</div>

</div>

</div>

<div id="concepts">

<div id="section-3" class="section">

## <a href="#section-3" class="section-number selfRef">3.</a> <a href="#name-concepts" class="section-name selfRef">Concepts</a>

This section describes the core components and lifecycle of Conditionally Released Digital Credentials at a conceptual level, prior to the normative data formats specified in <a href="#data-formats" class="auto internal xref">Section 4</a>.<a href="#section-3-1" class="pilcrow">¶</a>

<div id="key-provisioning">

<div id="section-3.1" class="section">

### <a href="#section-3.1" class="section-number selfRef">3.1.</a> <a href="#name-issuer-release-public-key-p" class="section-name selfRef">Issuer Release Public Key Provisioning</a>

During credential issuance (for example, using OpenID for Verifiable Credential Issuance \[<a href="#OpenID4VCI" class="cite xref">OpenID4VCI</a>\]), or via discoverable Issuer metadata, the Issuer provides the Credential Manager with an asymmetric public key termed the **Issuer Release Public Key** (`PK_I`), along with the Issuer's identifier (`iss`) and optionally a Credential Release Endpoint URL.<a href="#section-3.1-1" class="pilcrow">¶</a>

When an Issuer marks an issued credential as requiring conditional release, the Credential Manager stores the Issuer Release Public Key and Issuer metadata alongside the credential. To preserve Holder unlinkability, the Issuer <span class="bcp14">MUST</span> provision the same Issuer Release Public Key across a sufficiently large cohort of Holders and credentials (see <a href="#privacy-cohort" class="auto internal xref">Section 8.1</a>).<a href="#section-3.1-2" class="pilcrow">¶</a>

</div>

</div>

<div id="releasable-cred-and-token">

<div id="section-3.2" class="section">

### <a href="#section-3.2" class="section-number selfRef">3.2.</a> <a href="#name-releasable-credential-and-r" class="section-name selfRef">Releasable Credential and Release Token</a>

When a Verifier requests a credential presentation from the Credential Manager (e.g., via OpenID4VP over the W3C Digital Credentials API), the Credential Manager first constructs the standard presentation payload for the underlying credential format—including any Holder-selected disclosures and cryptographic Holder key binding (such as an SD-JWT+KB or an ISO mdoc `DeviceResponse` bound to the Verifier's `nonce` and `aud`/`client_id`).<a href="#section-3.2-1" class="pilcrow">¶</a>

Instead of returning this cleartext presentation directly to the Verifier, the Credential Manager performs a two-part wrapping operation:<a href="#section-3.2-2" class="pilcrow">¶</a>

1.  <div id="section-3.2-3.1">

    **Constructing the Releasable Credential**: The Credential Manager generates a fresh, single-use ephemeral symmetric AES Content Encryption Key (`K`). It encrypts the serialized Digital Credential presentation under `K` using an authenticated encryption with associated data (AEAD) algorithm. The resulting ciphertext structure is the **Releasable Credential**.<a href="#section-3.2-3.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-3.2-3.2">

    **Constructing the Release Token**: The Credential Manager encrypts the ephemeral AES key `K` using the Issuer's **Issuer Release Public Key** (`PK_I`). This encrypted key is packaged into a structure called the **Release Token**, which includes plaintext metadata identifying the Issuer (`iss`), the key identifier (`kid`), and optional release endpoint or credential type metadata so the Verifier knows where and how to redeem the token.<a href="#section-3.2-3.2.1" class="pilcrow">¶</a>

    </div>

</div>

</div>

<div id="crdc-concept">

<div id="section-3.3" class="section">

### <a href="#section-3.3" class="section-number selfRef">3.3.</a> <a href="#name-conditionally-released-digit" class="section-name selfRef">Conditionally Released Digital Credential (CRDC)</a>

The Credential Manager packages the **Release Token** and the **Releasable Credential** together into a composite envelope called the **Conditionally Released Digital Credential (CRDC)** and returns it to the Verifier as the Digital Credential response.<a href="#section-3.3-1" class="pilcrow">¶</a>

From the perspective of presentation protocols such as OpenID4VP, a Conditionally Released Digital Credential can be negotiated as a distinct credential format wrapper derived from the underlying format by appending the `-cr` suffix (for example, `dc+sd-jwt-cr` or `mso_mdoc-cr`).<a href="#section-3.3-2" class="pilcrow">¶</a>

</div>

</div>

<div id="redemption-concept">

<div id="section-3.4" class="section">

### <a href="#section-3.4" class="section-number selfRef">3.4.</a> <a href="#name-credential-release-redempti" class="section-name selfRef">Credential Release Redemption and Verification</a>

Upon receiving a Conditionally Released Digital Credential, the Verifier cannot immediately inspect or verify the underlying credential claims because they are encrypted within the Releasable Credential under the ephemeral AES key `K`. To unlock the credential, the Verifier performs the following steps:<a href="#section-3.4-1" class="pilcrow">¶</a>

1.  <div id="section-3.4-2.1">

    **Inspect Metadata**: The Verifier inspects the unencrypted metadata in the **Release Token** to identify the Issuer (and, via Issuer metadata discovery or an explicit claim, the Issuer's Credential Release Endpoint).<a href="#section-3.4-2.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-3.4-2.2">

    **Request Release from Issuer**: The Verifier sends the **Release Token** in an authenticated request to the Issuer's Credential Release Endpoint, requesting that the presentation key be released.<a href="#section-3.4-2.2.1" class="pilcrow">¶</a>

    </div>

3.  <div id="section-3.4-2.3">

    **Issuer Approval and Billing**: The Issuer authenticates the Verifier, verifies that the Verifier is authorized to receive presentations, and optionally records a billable event or deducts from the Verifier's quota. If approved, the Issuer uses its private key (`SK_I`) corresponding to the Issuer Release Public Key (`PK_I`) to decrypt the Release Token, recovering the one-time ephemeral AES key `K`, and returns `K` to the Verifier over the TLS-protected channel.<a href="#section-3.4-2.3.1" class="pilcrow">¶</a>

    </div>

4.  <div id="section-3.4-2.4">

    **Decrypt Releasable Credential**: The Verifier uses the returned AES key `K` to decrypt the **Releasable Credential**, revealing the underlying Digital Credential presentation.<a href="#section-3.4-2.4.1" class="pilcrow">¶</a>

    </div>

5.  <div id="section-3.4-2.5">

    **Standard Credential Validation**: The Verifier validates the underlying Digital Credential presentation exactly as specified by its native format (e.g., verifying the Issuer signature, Holder key binding, disclosures, validity period, and Verifier nonce/audience binding).<a href="#section-3.4-2.5.1" class="pilcrow">¶</a>

    </div>

Crucially, during Step 2 and Step 3, the Issuer only ever sees the Verifier's identity and the **Release Token** (which wraps a random ephemeral key `K` generated by the Credential Manager at presentation time). The Issuer never sees the Releasable Credential, any credential claims, or any identifier tied to the Holder or issuance transaction.<a href="#section-3.4-3" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="data-formats">

<div id="section-4" class="section">

## <a href="#section-4" class="section-number selfRef">4.</a> <a href="#name-data-formats-and-cryptograp" class="section-name selfRef">Data Formats and Cryptographic Wrapper</a>

<div id="issuer-metadata">

<div id="section-4.1" class="section">

### <a href="#section-4.1" class="section-number selfRef">4.1.</a> <a href="#name-issuer-metadata-parameters" class="section-name selfRef">Issuer Metadata Parameters</a>

Issuers supporting Conditionally Released Digital Credentials advertise their conditional release parameters in their Credential Issuer Metadata (for example, the Credential Issuer Metadata document defined in \[<a href="#OpenID4VCI" class="cite xref">OpenID4VCI</a>\] or OAuth Authorization Server Metadata \[<a href="#RFC8414" class="cite xref">RFC8414</a>\]).<a href="#section-4.1-1" class="pilcrow">¶</a>

The following metadata parameters are defined:<a href="#section-4.1-2" class="pilcrow">¶</a>

<span class="break"></span>

`credential_release_endpoint`:  
<span class="bcp14">REQUIRED</span>. URL of the Issuer's Credential Release Endpoint where Verifiers send Release Tokens to obtain the decrypted ephemeral key. This URL <span class="bcp14">MUST</span> use the `https` scheme.<a href="#section-4.1-3.2.1" class="pilcrow">¶</a>

`credential_release_encryption_jwks`:  
<span class="bcp14">REQUIRED</span> (unless `credential_release_encryption_jwks_uri` is provided). A JSON Web Key Set (JWKS) \[<a href="#RFC7517" class="cite xref">RFC7517</a>\] containing one or more **Issuer Release Public Keys** used by Credential Managers to encrypt ephemeral Content Encryption Keys into Release Tokens. Each JWK <span class="bcp14">MUST</span> contain a `kid` (Key ID), `kty` (Key Type), `use` set to `"enc"`, and an `alg` (Algorithm) parameter (e.g., `"ECDH-ES"`, `"ECDH-ES+A256KW"`, or `"RSA-OAEP-256"`).<a href="#section-4.1-3.4.1" class="pilcrow">¶</a>

`credential_release_encryption_jwks_uri`:  
<span class="bcp14">OPTIONAL</span>. An HTTPS URL referencing a JWKS document containing the Issuer's **Issuer Release Public Keys**.<a href="#section-4.1-3.6.1" class="pilcrow">¶</a>

`credential_release_required`:  
<span class="bcp14">OPTIONAL</span>. A boolean value inside a specific credential configuration object in Issuer metadata indicating whether presentations of this credential <span class="bcp14">MUST</span> be wrapped as a Conditionally Released Digital Credential. Defaults to `false` if omitted.<a href="#section-4.1-3.8.1" class="pilcrow">¶</a>

</div>

</div>

<div id="releasable-credential-format">

<div id="section-4.2" class="section">

### <a href="#section-4.2" class="section-number selfRef">4.2.</a> <a href="#name-releasable-credential-forma" class="section-name selfRef">Releasable Credential Format</a>

The **Releasable Credential** represents the underlying Digital Credential presentation encrypted under the one-time ephemeral AES key `K` (the Content Encryption Key, or CEK).<a href="#section-4.2-1" class="pilcrow">¶</a>

To construct the Releasable Credential:<a href="#section-4.2-2" class="pilcrow">¶</a>

1.  <div id="section-4.2-3.1">

    The Credential Manager produces the standard Digital Credential presentation in its native format (e.g., a UTF-8 string for an SD-JWT+KB compact serialization, or raw CBOR bytes for an ISO/IEC 18013-5 `DeviceResponse`).<a href="#section-4.2-3.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-4.2-3.2">

    The Credential Manager generates a fresh, cryptographically random symmetric AES Content Encryption Key `K` of length 256 bits (32 octets) (or 128 bits if `A128GCM` is used) and a random 96-bit (12-octet) Initialization Vector (`IV`).<a href="#section-4.2-3.2.1" class="pilcrow">¶</a>

    </div>

3.  <div id="section-4.2-3.3">

    The Credential Manager encrypts the raw octets of the underlying Digital Credential presentation using AES-GCM (`A256GCM` <span class="bcp14">REQUIRED</span>, `A128GCM` <span class="bcp14">OPTIONAL</span>) \[<a href="#RFC7518" class="cite xref">RFC7518</a>\].<a href="#section-4.2-3.3.1" class="pilcrow">¶</a>

    </div>

4.  <div id="section-4.2-3.4">

    To cryptographically bind the Releasable Credential to the Release Token and prevent mix-and-match substitution attacks, the ASCII bytes of the compact **Release Token** (defined in <a href="#release-token-format" class="auto internal xref">Section 4.3</a>) <span class="bcp14">MUST</span> be passed as the Additional Authenticated Data (`AAD`) to the AES-GCM encryption operation.<a href="#section-4.2-3.4.1" class="pilcrow">¶</a>

    </div>

The `releasable_credential` is represented as a JSON object with the following members:<a href="#section-4.2-4" class="pilcrow">¶</a>

<span class="break"></span>

`enc`:  
<span class="bcp14">REQUIRED</span>. The symmetric content encryption algorithm used to encrypt the underlying credential presentation. <span class="bcp14">MUST</span> be a registered JWE `"enc"` algorithm name from \[<a href="#RFC7518" class="cite xref">RFC7518</a>\], with `"A256GCM"` as the default and Mandatory-to-Implement (MTI) algorithm.<a href="#section-4.2-5.2.1" class="pilcrow">¶</a>

`iv`:  
<span class="bcp14">REQUIRED</span>. The base64url-encoded Initialization Vector (nonce) used for the AEAD encryption (12 octets for AES-GCM).<a href="#section-4.2-5.4.1" class="pilcrow">¶</a>

`ciphertext`:  
<span class="bcp14">REQUIRED</span>. The base64url-encoded AEAD ciphertext of the underlying Digital Credential presentation.<a href="#section-4.2-5.6.1" class="pilcrow">¶</a>

`tag`:  
<span class="bcp14">REQUIRED</span>. The base64url-encoded Authentication Tag produced by the AEAD encryption (16 octets for AES-GCM).<a href="#section-4.2-5.8.1" class="pilcrow">¶</a>

</div>

</div>

<div id="release-token-format">

<div id="section-4.3" class="section">

### <a href="#section-4.3" class="section-number selfRef">4.3.</a> <a href="#name-release-token-format" class="section-name selfRef">Release Token Format</a>

The **Release Token** encapsulates the one-time ephemeral AES key `K` encrypted under the **Issuer Release Public Key**, along with the Issuer metadata needed by the Verifier to locate the Issuer and redeem the token.<a href="#section-4.3-1" class="pilcrow">¶</a>

The Release Token is serialized as a JSON Web Encryption (JWE) Compact Serialization string \[<a href="#RFC7516" class="cite xref">RFC7516</a>\] whose plaintext payload contains the ephemeral AES key `K`.<a href="#section-4.3-2" class="pilcrow">¶</a>

<div id="release-token-header">

<div id="section-4.3.1" class="section">

#### <a href="#section-4.3.1" class="section-number selfRef">4.3.1.</a> <a href="#name-release-token-jwe-protected" class="section-name selfRef">Release Token JWE Protected Header</a>

The JWE Protected Header of the Release Token <span class="bcp14">MUST</span> contain the following parameters:<a href="#section-4.3.1-1" class="pilcrow">¶</a>

<span class="break"></span>

`alg`:  
<span class="bcp14">REQUIRED</span>. The asymmetric key encryption / key agreement algorithm used to encrypt the ephemeral key `K` under the Issuer Release Public Key (e.g., `"ECDH-ES+A256KW"`, `"ECDH-ES"`, or `"RSA-OAEP-256"`), or an HPKE-based JWE algorithm \[<a href="#RFC9180" class="cite xref">RFC9180</a>\]. `"ECDH-ES+A256KW"` with curve `"P-256"` is Mandatory-to-Implement (MTI).<a href="#section-4.3.1-2.2.1" class="pilcrow">¶</a>

`enc`:  
<span class="bcp14">REQUIRED</span>. The content encryption algorithm used inside the JWE to encrypt the payload containing `K` (e.g., `"A256GCM"`).<a href="#section-4.3.1-2.4.1" class="pilcrow">¶</a>

`typ`:  
<span class="bcp14">REQUIRED</span>. Media type of the Release Token. <span class="bcp14">MUST</span> be `"crdc-release+jwe"`.<a href="#section-4.3.1-2.6.1" class="pilcrow">¶</a>

`kid`:  
<span class="bcp14">REQUIRED</span>. The Key Identifier of the **Issuer Release Public Key** used to encrypt the Release Token.<a href="#section-4.3.1-2.8.1" class="pilcrow">¶</a>

`iss`:  
<span class="bcp14">REQUIRED</span>. The Issuer identifier (an HTTPS URL with no query or fragment components) identifying the Issuer that issued the underlying Digital Credential and that can decrypt this Release Token.<a href="#section-4.3.1-2.10.1" class="pilcrow">¶</a>

`cr_ep`:  
<span class="bcp14">OPTIONAL</span>. The HTTPS URL of the Issuer's Credential Release Endpoint (`credential_release_endpoint`). Including `cr_ep` allows the Verifier to contact the Release Endpoint directly, though the Verifier <span class="bcp14">SHOULD</span> validate it against discovered Issuer metadata for `iss`.<a href="#section-4.3.1-2.12.1" class="pilcrow">¶</a>

Note: When using `"alg": "ECDH-ES"` or `"ECDH-ES+A256KW"`, the JWE Protected Header also includes the standard `"epk"` (Ephemeral Public Key) parameter generated by the Credential Manager for this single presentation.<a href="#section-4.3.1-3" class="pilcrow">¶</a>

</div>

</div>

<div id="release-token-payload">

<div id="section-4.3.2" class="section">

#### <a href="#section-4.3.2" class="section-number selfRef">4.3.2.</a> <a href="#name-release-token-plaintext-pay" class="section-name selfRef">Release Token Plaintext Payload</a>

The plaintext encrypted inside the Release Token JWE <span class="bcp14">MUST</span> be a UTF-8 encoded JSON object containing at least the following members:<a href="#section-4.3.2-1" class="pilcrow">¶</a>

<span class="break"></span>

`k`:  
<span class="bcp14">REQUIRED</span>. The base64url-encoded raw octets of the one-time ephemeral AES Content Encryption Key `K` used to encrypt the `releasable_credential`.<a href="#section-4.3.2-2.2.1" class="pilcrow">¶</a>

`enc`:  
<span class="bcp14">REQUIRED</span>. The symmetric algorithm identifier (e.g., `"A256GCM"`) with which `K` is used.<a href="#section-4.3.2-2.4.1" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="crdc-envelope">

<div id="section-4.4" class="section">

### <a href="#section-4.4" class="section-number selfRef">4.4.</a> <a href="#name-conditionally-released-digita" class="section-name selfRef">Conditionally Released Digital Credential (CRDC) Envelope</a>

The **Conditionally Released Digital Credential (CRDC)** is the top-level JSON object returned by the Credential Manager to the Verifier in the presentation response. It contains the following fields:<a href="#section-4.4-1" class="pilcrow">¶</a>

<span class="break"></span>

`format`:  
<span class="bcp14">REQUIRED</span>. The presentation format identifier indicating a conditionally released credential (e.g., `"dc+sd-jwt-cr"` or `"mso_mdoc-cr"`), or the underlying credential format identifier when the outer protocol already signals the `-cr` format.<a href="#section-4.4-2.2.1" class="pilcrow">¶</a>

`release_token`:  
<span class="bcp14">REQUIRED</span>. The JWE Compact Serialization string representing the Release Token (as defined in <a href="#release-token-format" class="auto internal xref">Section 4.3</a>).<a href="#section-4.4-2.4.1" class="pilcrow">¶</a>

`releasable_credential`:  
<span class="bcp14">REQUIRED</span>. The JSON object representing the encrypted Digital Credential presentation (as defined in <a href="#releasable-credential-format" class="auto internal xref">Section 4.2</a>).<a href="#section-4.4-2.6.1" class="pilcrow">¶</a>

Example CRDC structure:<a href="#section-4.4-3" class="pilcrow">¶</a>

<div id="section-4.4-4" class="lang-json sourcecode">

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

<a href="#section-4.4-4" class="pilcrow">¶</a>

</div>

</div>

</div>

<div id="format-identifiers">

<div id="section-4.5" class="section">

### <a href="#section-4.5" class="section-number selfRef">4.5.</a> <a href="#name-presentation-format-identif" class="section-name selfRef">Presentation Format Identifiers (<code>-cr</code> Suffix Convention)</a>

When used with presentation protocols that negotiate credential formats (such as OpenID4VP \[<a href="#OpenID4VP" class="cite xref">OpenID4VP</a>\] and the W3C Digital Credentials API \[<a href="#W3C.DC-API" class="cite xref">W3C.DC-API</a>\]), a Conditionally Released Digital Credential format identifier is constructed by appending the suffix **`-cr`** to the underlying credential format identifier:<a href="#section-4.5-1" class="pilcrow">¶</a>

| Underlying Credential Format                                                      | Underlying Format Identifier | Conditionally Released Format Identifier |
|-----------------------------------------------------------------------------------|------------------------------|------------------------------------------|
| SD-JWT VC \[<a href="#SD-JWT-VC" class="cite xref">SD-JWT-VC</a>\]                | `dc+sd-jwt` (or `vc+sd-jwt`) | `dc+sd-jwt-cr` (or `vc+sd-jwt-cr`)       |
| Bare SD-JWT \[<a href="#RFC9901" class="cite xref">RFC9901</a>\]                  | `sd-jwt`                     | `sd-jwt-cr`                              |
| ISO/IEC 18013-5 mdoc \[<a href="#ISO.18013-5" class="cite xref">ISO.18013-5</a>\] | `mso_mdoc`                   | `mso_mdoc-cr`                            |
| W3C Verifiable Credentials                                                        | `jwt_vc_json` / `ldp_vc`     | `jwt_vc_json-cr` / `ldp_vc-cr`           |

<a href="#table-1" class="selfRef">Table 1</a>

When a Verifier includes a `-cr` format identifier in its presentation request (or indicates support for conditional release), a Credential Manager holding a credential that requires conditional release returns the CRDC JSON envelope defined in <a href="#crdc-envelope" class="auto internal xref">Section 4.4</a>.<a href="#section-4.5-3" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="protocol-flow">

<div id="section-5" class="section">

## <a href="#section-5" class="section-number selfRef">5.</a> <a href="#name-protocol-flow-and-processin" class="section-name selfRef">Protocol Flow and Processing Rules</a>

<div id="issuance-flow">

<div id="section-5.1" class="section">

### <a href="#section-5.1" class="section-number selfRef">5.1.</a> <a href="#name-issuance-and-key-discovery" class="section-name selfRef">Issuance and Key Discovery</a>

1.  <div id="section-5.1-1.1">

    **Issuer Key Generation**: The Issuer generates one or more asymmetric key pairs (`PK_I`, `SK_I`) designated as **Issuer Release Keys** and publishes the public keys (`PK_I`) in `credential_release_encryption_jwks` alongside its `credential_release_endpoint` in its Issuer Metadata.<a href="#section-5.1-1.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-5.1-1.2">

    **Provisioning to Credential Manager**: During credential issuance (e.g., via OpenID4VCI), the Credential Manager retrieves and caches the Issuer's `credential_release_encryption_jwks` and `iss` identifier associated with the issued Digital Credential. The Credential Manager <span class="bcp14">SHOULD</span> periodically refresh the Issuer's release public keys in accordance with HTTP cache headers, using anonymous network transport (e.g., Oblivious HTTP \[<a href="#RFC9458" class="cite xref">RFC9458</a>\] or proxy) if key rotation fetches occur post-issuance.<a href="#section-5.1-1.2.1" class="pilcrow">¶</a>

    </div>

</div>

</div>

<div id="presentation-flow">

<div id="section-5.2" class="section">

### <a href="#section-5.2" class="section-number selfRef">5.2.</a> <a href="#name-presentation-by-the-credent" class="section-name selfRef">Presentation by the Credential Manager</a>

When the Credential Manager receives a presentation request from a Verifier for a credential configured for conditional release, the Credential Manager <span class="bcp14">MUST</span> perform the following steps:<a href="#section-5.2-1" class="pilcrow">¶</a>

1.  <div id="section-5.2-2.1">

    **Generate Underlying Presentation**: Construct the native Digital Credential presentation (`DC_pres`) according to the rules of the underlying credential format and presentation protocol. Crucially, any cryptographic Holder Key Binding (e.g., the Key Binding JWT in SD-JWT+KB or `DeviceAuth` in ISO mdoc) <span class="bcp14">MUST</span> bind to the Verifier's `nonce` and `aud`/`client_id` inside the inner `DC_pres` prior to encryption.<a href="#section-5.2-2.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-5.2-2.2">

    **Generate Ephemeral AES Key**: Generate a fresh, cryptographically random 256-bit AES key `K` and a 96-bit random `IV` using a cryptographically secure pseudorandom number generator (CSPRNG) \[<a href="#RFC4086" class="cite xref">RFC4086</a>\]. `K` <span class="bcp14">MUST NOT</span> be reused across presentations.<a href="#section-5.2-2.2.1" class="pilcrow">¶</a>

    </div>

3.  <div id="section-5.2-2.3">

    **Create Release Token**:<a href="#section-5.2-2.3.1" class="pilcrow">¶</a>

    - <div id="section-5.2-2.3.2.1">

      Construct the Release Token plaintext JSON object `{"k": base64url(K), "enc": "A256GCM"}`.<a href="#section-5.2-2.3.2.1.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.2-2.3.2.2">

      Construct the JWE Protected Header containing `"alg"`, `"enc"`, `"typ": "crdc-release+jwe"`, `"kid"`, and `"iss"` (set to the Issuer's identifier URL).<a href="#section-5.2-2.3.2.2.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.2-2.3.2.3">

      Encrypt the plaintext under the Issuer's **Issuer Release Public Key** (`PK_I`) to produce the JWE Compact Serialization string `release_token`.<a href="#section-5.2-2.3.2.3.1" class="pilcrow">¶</a>

      </div>

    </div>

4.  <div id="section-5.2-2.4">

    **Create Releasable Credential**:<a href="#section-5.2-2.4.1" class="pilcrow">¶</a>

    - <div id="section-5.2-2.4.2.1">

      Encrypt the raw octets of `DC_pres` using AES-256-GCM with key `K`, initialization vector `IV`, and Additional Authenticated Data (`AAD`) set to the ASCII bytes of `release_token`.<a href="#section-5.2-2.4.2.1.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.2-2.4.2.2">

      Construct the `releasable_credential` JSON object containing `enc`, `iv`, `ciphertext`, and `tag`.<a href="#section-5.2-2.4.2.2.1" class="pilcrow">¶</a>

      </div>

    </div>

5.  <div id="section-5.2-2.5">

    **Return CRDC**: Package `format`, `release_token`, and `releasable_credential` into the CRDC JSON object and return it to the Verifier via the presentation protocol (e.g., over the W3C Digital Credentials API).<a href="#section-5.2-2.5.1" class="pilcrow">¶</a>

    </div>

</div>

</div>

<div id="redemption-flow">

<div id="section-5.3" class="section">

### <a href="#section-5.3" class="section-number selfRef">5.3.</a> <a href="#name-release-token-redemption-ve" class="section-name selfRef">Release Token Redemption (Verifier to Issuer)</a>

When a Verifier receives a Conditionally Released Digital Credential (`CRDC`), it <span class="bcp14">MUST</span> perform the following steps to obtain the ephemeral decryption key:<a href="#section-5.3-1" class="pilcrow">¶</a>

1.  <div id="section-5.3-2.1">

    **Parse Release Token Header**: Base64url-decode the JWE Protected Header of `release_token` and extract the `iss` claim (and optional `cr_ep` claim).<a href="#section-5.3-2.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-5.3-2.2">

    **Resolve Release Endpoint**: Determine the Issuer's `credential_release_endpoint` from the Verifier's pre-configured trust store or by fetching the Issuer Metadata from `iss` (e.g., `/.well-known/openid-credential-issuer`). The Verifier <span class="bcp14">MUST</span> verify that `iss` is a trusted Issuer before sending a request.<a href="#section-5.3-2.2.1" class="pilcrow">¶</a>

    </div>

3.  <div id="section-5.3-2.3">

    **Send Release Request**: Send an HTTP `POST` request to the Issuer's `credential_release_endpoint` with content type `application/json`. The Verifier <span class="bcp14">MUST</span> authenticate itself to the Issuer (for example, using an OAuth 2.0 Bearer access token \[<a href="#RFC6750" class="cite xref">RFC6750</a>\], mutual TLS \[<a href="#RFC8705" class="cite xref">RFC8705</a>\], or `private_key_jwt` client authentication). The request body <span class="bcp14">MUST</span> be a JSON object containing:<a href="#section-5.3-2.3.1" class="pilcrow">¶</a>

    - <div id="section-5.3-2.3.2.1">

      `release_token`: <span class="bcp14">REQUIRED</span>. The exact `release_token` string received in the CRDC.<a href="#section-5.3-2.3.2.1.1" class="pilcrow">¶</a>

      </div>

    </div>

Example Release Request:<a href="#section-5.3-3" class="pilcrow">¶</a>

<div id="section-5.3-4" class="lang-http sourcecode">

    POST /release HTTP/1.1
    Host: issuer.example.com
    Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
    Content-Type: application/json

    {
      "release_token": "eyJhbGciOiJFQ0RILUVTK0EyNTZLVyIsImVuYyI6IkEyNTZHQ00i..."
    }

<a href="#section-5.3-4" class="pilcrow">¶</a>

</div>

1.  <div id="section-5.3-5.1">

    **Issuer Processing and Approval**: Upon receiving the request at `credential_release_endpoint`, the Issuer:<a href="#section-5.3-5.1.1" class="pilcrow">¶</a>

    - <div id="section-5.3-5.1.2.1">

      Authenticates the Verifier and verifies that the Verifier has an active commercial/trust relationship and sufficient quota or billing authorization.<a href="#section-5.3-5.1.2.1.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.3-5.1.2.2">

      Looks up the private key (`SK_I`) corresponding to the `kid` in the JWE Protected Header of `release_token`.<a href="#section-5.3-5.1.2.2.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.3-5.1.2.3">

      Decrypts and verifies the integrity of `release_token` using `SK_I`. If decryption fails, the Issuer <span class="bcp14">MUST</span> return an HTTP `400 Bad Request` error with error code `"invalid_release_token"` and <span class="bcp14">MUST NOT</span> bill the Verifier.<a href="#section-5.3-5.1.2.3.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.3-5.1.2.4">

      Records the billable transaction against the Verifier's account.<a href="#section-5.3-5.1.2.4.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.3-5.1.2.5">

      Returns an HTTP `200 OK` response with `Content-Type: application/json` and `Cache-Control: no-store` containing the released key object (`k` and `enc`).<a href="#section-5.3-5.1.2.5.1" class="pilcrow">¶</a>

      </div>

    </div>

Example Release Response:<a href="#section-5.3-6" class="pilcrow">¶</a>

<div id="section-5.3-7" class="lang-http sourcecode">

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
      "k": "f83j2k1l0m9n8b7v6c5x4z3a2s1d0f9g8h7j6k5l4m3",
      "enc": "A256GCM"
    }

<a href="#section-5.3-7" class="pilcrow">¶</a>

</div>

</div>

</div>

<div id="verification-flow">

<div id="section-5.4" class="section">

### <a href="#section-5.4" class="section-number selfRef">5.4.</a> <a href="#name-unwrapping-and-verification" class="section-name selfRef">Unwrapping and Verification by the Verifier</a>

Upon receiving the HTTP `200 OK` response from the Issuer's Credential Release Endpoint, the Verifier <span class="bcp14">MUST</span> perform the following steps:<a href="#section-5.4-1" class="pilcrow">¶</a>

1.  <div id="section-5.4-2.1">

    **Extract Ephemeral Key**: Base64url-decode the `k` parameter from the Issuer's response to obtain the symmetric AES key `K`.<a href="#section-5.4-2.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-5.4-2.2">

    **Decrypt Releasable Credential**: Using key `K`, the `iv`, `ciphertext`, and `tag` from `releasable_credential`, and the ASCII bytes of `release_token` as the `AAD`, perform AES-GCM authenticated decryption. If decryption or tag verification fails, the Verifier <span class="bcp14">MUST</span> abort processing and reject the presentation.<a href="#section-5.4-2.2.1" class="pilcrow">¶</a>

    </div>

3.  <div id="section-5.4-2.3">

    **Validate Underlying Digital Credential**: Parse the decrypted plaintext as the underlying Digital Credential presentation (`DC_pres`, e.g., an SD-JWT+KB or ISO mdoc `DeviceResponse`) and perform all standard validation checks required by the underlying format and presentation protocol, including:<a href="#section-5.4-2.3.1" class="pilcrow">¶</a>

    - <div id="section-5.4-2.3.2.1">

      Verifying the Issuer's cryptographic signature on the credential using the Issuer's signing public key.<a href="#section-5.4-2.3.2.1.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.4-2.3.2.2">

      Verifying that the Issuer of the inner credential matches the `iss` in the outer `release_token` header.<a href="#section-5.4-2.3.2.2.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.4-2.3.2.3">

      Verifying Holder Key Binding (`kb+jwt` or `DeviceAuth`), ensuring that the `nonce` and `aud`/`client_id` match the Verifier's presentation request.<a href="#section-5.4-2.3.2.3.1" class="pilcrow">¶</a>

      </div>

    - <div id="section-5.4-2.3.2.4">

      Verifying selectively disclosed claims and credential validity/revocation status.<a href="#section-5.4-2.3.2.4.1" class="pilcrow">¶</a>

      </div>

    </div>

</div>

</div>

</div>

</div>

<div id="examples">

<div id="section-6" class="section">

## <a href="#section-6" class="section-number selfRef">6.</a> <a href="#name-examples" class="section-name selfRef">Examples</a>

<div id="example-metadata">

<div id="section-6.1" class="section">

### <a href="#section-6.1" class="section-number selfRef">6.1.</a> <a href="#name-issuer-metadata-example" class="section-name selfRef">Issuer Metadata Example</a>

The following non-normative example shows an Issuer publishing its `credential_release_endpoint` and `credential_release_encryption_jwks` inside its Credential Issuer Metadata:<a href="#section-6.1-1" class="pilcrow">¶</a>

<div id="section-6.1-2" class="lang-json sourcecode">

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

<a href="#section-6.1-2" class="pilcrow">¶</a>

</div>

</div>

</div>

<div id="example-sd-jwt-cr">

<div id="section-6.2" class="section">

### <a href="#section-6.2" class="section-number selfRef">6.2.</a> <a href="#name-conditionally-released-sd-j" class="section-name selfRef">Conditionally Released SD-JWT (<code>dc+sd-jwt-cr</code>)</a>

When wrapping an SD-JWT+KB presentation (`<Issuer-signed JWT>~<Disclosure 1>~...~<KB-JWT>`), the Credential Manager encrypts the UTF-8 octets of the complete SD-JWT+KB string into `releasable_credential` and returns:<a href="#section-6.2-1" class="pilcrow">¶</a>

<div id="section-6.2-2" class="lang-json sourcecode">

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

<a href="#section-6.2-2" class="pilcrow">¶</a>

</div>

</div>

</div>

<div id="example-mdoc-cr">

<div id="section-6.3" class="section">

### <a href="#section-6.3" class="section-number selfRef">6.3.</a> <a href="#name-conditionally-released-iso-" class="section-name selfRef">Conditionally Released ISO mdoc (<code>mso_mdoc-cr</code>)</a>

When wrapping an ISO/IEC 18013-5 `DeviceResponse` CBOR structure, the Credential Manager encrypts the raw CBOR octets of `DeviceResponse` into `releasable_credential` and sets `"format": "mso_mdoc-cr"`. Upon decrypting `releasable_credential.ciphertext`, the Verifier obtains the exact CBOR bytes of `DeviceResponse` and validates the `IssuerAuth` COSE_Sign1 and `DeviceAuth` structures as normal.<a href="#section-6.3-1" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="security-considerations">

<div id="section-7" class="section">

## <a href="#section-7" class="section-number selfRef">7.</a> <a href="#name-security-considerations" class="section-name selfRef">Security Considerations</a>

<div id="sec-entropy">

<div id="section-7.1" class="section">

### <a href="#section-7.1" class="section-number selfRef">7.1.</a> <a href="#name-ephemeral-key-entropy-and-n" class="section-name selfRef">Ephemeral Key Entropy and Nonce Uniqueness</a>

The security of the Releasable Credential relies on the unpredictability of the one-time ephemeral AES key `K`. Credential Managers <span class="bcp14">MUST</span> generate a fresh, cryptographically random key `K` and `IV` for every presentation. Reusing `K` across presentations would allow a Verifier that paid for one presentation to decrypt another presentation without Issuer approval, and would enable correlation across presentations.<a href="#section-7.1-1" class="pilcrow">¶</a>

</div>

</div>

<div id="sec-binding">

<div id="section-7.2" class="section">

### <a href="#section-7.2" class="section-number selfRef">7.2.</a> <a href="#name-binding-between-releasable-" class="section-name selfRef">Binding Between Releasable Credential and Release Token</a>

By passing the ASCII representation of `release_token` as Additional Authenticated Data (`AAD`) when encrypting `releasable_credential`, the two structures are cryptographically bound. An attacker cannot strip a `release_token` from one CRDC and attach it to a different `releasable_credential` without causing AEAD tag verification to fail.<a href="#section-7.2-1" class="pilcrow">¶</a>

</div>

</div>

<div id="sec-inner-issuer">

<div id="section-7.3" class="section">

### <a href="#section-7.3" class="section-number selfRef">7.3.</a> <a href="#name-inner-issuer-verification-a" class="section-name selfRef">Inner Issuer Verification Against Outer Release Token Issuer</a>

A malicious Issuer `M` could attempt to wrap a credential issued by honest Issuer `H` under `M`'s Release Public Key in order to collect fees meant for `H`. To prevent this, as specified in <a href="#verification-flow" class="auto internal xref">Section 5.4</a>, the Verifier <span class="bcp14">MUST</span> verify after decrypting the Releasable Credential that the cryptographic Issuer of the inner Digital Credential matches the `iss` claimed in the outer `release_token` JWE Protected Header.<a href="#section-7.3-1" class="pilcrow">¶</a>

</div>

</div>

<div id="sec-holder-key-binding">

<div id="section-7.4" class="section">

### <a href="#section-7.4" class="section-number selfRef">7.4.</a> <a href="#name-inner-holder-key-binding" class="section-name selfRef">Inner Holder Key Binding</a>

The CRDC wrapper provides conditional confidentiality of the presentation until Issuer approval, but does not replace the inner credential's Holder Key Binding. The inner Digital Credential presentation (e.g., SD-JWT+KB or mdoc `DeviceResponse`) <span class="bcp14">MUST</span> still include cryptographic Holder Key Binding over the Verifier's `nonce` and `aud`/`client_id`. This ensures that even after decryption, the Verifier has cryptographic proof that the Holder consented to present the credential to that specific Verifier in that specific session.<a href="#section-7.4-1" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="privacy-considerations">

<div id="section-8" class="section">

## <a href="#section-8" class="section-number selfRef">8.</a> <a href="#name-privacy-considerations" class="section-name selfRef">Privacy Considerations</a>

<div id="privacy-cohort">

<div id="section-8.1" class="section">

### <a href="#section-8.1" class="section-number selfRef">8.1.</a> <a href="#name-unlinkability-against-the-i" class="section-name selfRef">Unlinkability Against the Issuer (Shared Key Cohort Requirements)</a>

The primary privacy guarantee of Conditionally Released Digital Credentials is that the Issuer can authorize and bill a Verifier for a presentation without learning which Holder's credential is being presented.<a href="#section-8.1-1" class="pilcrow">¶</a>

To uphold this guarantee:<a href="#section-8.1-2" class="pilcrow">¶</a>

1.  <div id="section-8.1-3.1">

    **No Per-Holder Release Keys**: The Issuer <span class="bcp14">MUST NOT</span> provision unique or fine-grained **Issuer Release Public Keys** (`PK_I` / `kid`) to individual Holders or small cohorts of Holders. If an Issuer assigned a distinct `PK_I` to each Holder, the `kid` (or successful trial decryption) at the Release Endpoint would immediately reveal which Holder is presenting their credential to the Verifier. Issuers <span class="bcp14">MUST</span> share each Issuer Release Public Key across a large anonymity cohort (e.g., all credentials of a given type issued within a broad time window).<a href="#section-8.1-3.1.1" class="pilcrow">¶</a>

    </div>

2.  <div id="section-8.1-3.2">

    **No Holder-Identifying Metadata in the Release Token**: The Credential Manager <span class="bcp14">MUST NOT</span> include any Holder identifier, credential serial number, issuance timestamp, or static salt inside either the JWE Protected Header or the encrypted payload of the `release_token`. Both the ephemeral asymmetric key (`epk`) used in JWE key agreement and the ephemeral symmetric key (`K`) inside the JWE payload <span class="bcp14">MUST</span> be freshly generated at presentation time.<a href="#section-8.1-3.2.1" class="pilcrow">¶</a>

    </div>

</div>

</div>

<div id="privacy-confidentiality">

<div id="section-8.2" class="section">

### <a href="#section-8.2" class="section-number selfRef">8.2.</a> <a href="#name-confidentiality-of-claims-a" class="section-name selfRef">Confidentiality of Claims Against the Issuer</a>

Because the Verifier only sends the `release_token` to the Issuer's Release Endpoint and never sends the `releasable_credential`, the Issuer never sees the underlying credential, the Holder's selectively disclosed claims, or the Verifier's session nonce.<a href="#section-8.2-1" class="pilcrow">¶</a>

</div>

</div>

<div id="privacy-timing">

<div id="section-8.3" class="section">

### <a href="#section-8.3" class="section-number selfRef">8.3.</a> <a href="#name-timing-and-network-correlat" class="section-name selfRef">Timing and Network Correlation Mitigations</a>

If an Issuer issues a credential to a Holder and the Holder immediately presents that credential to a Verifier milliseconds later, the Issuer could attempt to correlate the issuance event with the Verifier's Release Token redemption by timing alone. Credential Managers <span class="bcp14">SHOULD</span> pre-provision credentials and Issuer Release Public Keys ahead of presentation time, or introduce jitter when issuance and presentation occur back-to-back.<a href="#section-8.3-1" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="iana-considerations">

<div id="section-9" class="section">

## <a href="#section-9" class="section-number selfRef">9.</a> <a href="#name-iana-considerations" class="section-name selfRef">IANA Considerations</a>

<div id="iana-metadata">

<div id="section-9.1" class="section">

### <a href="#section-9.1" class="section-number selfRef">9.1.</a> <a href="#name-oauth-authorization-server-" class="section-name selfRef">OAuth Authorization Server / Credential Issuer Metadata Registry</a>

This specification requests registration of the following metadata parameters:<a href="#section-9.1-1" class="pilcrow">¶</a>

- <div id="section-9.1-2.1">

  `credential_release_endpoint`: URL of the Issuer's Credential Release Endpoint.<a href="#section-9.1-2.1.1" class="pilcrow">¶</a>

  </div>

- <div id="section-9.1-2.2">

  `credential_release_encryption_jwks`: JWKS containing the Issuer Release Public Keys.<a href="#section-9.1-2.2.1" class="pilcrow">¶</a>

  </div>

- <div id="section-9.1-2.3">

  `credential_release_encryption_jwks_uri`: URL of the JWKS containing the Issuer Release Public Keys.<a href="#section-9.1-2.3.1" class="pilcrow">¶</a>

  </div>

</div>

</div>

<div id="iana-media-type">

<div id="section-9.2" class="section">

### <a href="#section-9.2" class="section-number selfRef">9.2.</a> <a href="#name-media-type-registration" class="section-name selfRef">Media Type Registration</a>

This section registers the `"application/crdc-release+jwe"` media type (`typ` shorthand `"crdc-release+jwe"`) in the IANA "Media Types" registry to identify a Release Token JWE.<a href="#section-9.2-1" class="pilcrow">¶</a>

</div>

</div>

</div>

</div>

<div id="sec-combined-references">

<div id="section-10" class="section">

## <a href="#section-10" class="section-number selfRef">10.</a> <a href="#name-references" class="section-name selfRef">References</a>

<div id="sec-normative-references">

<div id="section-10.1" class="section">

### <a href="#section-10.1" class="section-number selfRef">10.1.</a> <a href="#name-normative-references" class="section-name selfRef">Normative References</a>

\[RFC2119\]  
<span class="refAuthor">Bradner, S.</span>, <span class="refTitle">"Key words for use in RFCs to Indicate Requirement Levels"</span>, <span class="seriesInfo">BCP 14</span>, <span class="seriesInfo">RFC 2119</span>, <span class="seriesInfo">DOI 10.17487/RFC2119</span>, March 1997, \<<https://www.rfc-editor.org/rfc/rfc2119>\>.

\[RFC4086\]  
<span class="refAuthor">Eastlake 3rd, D.</span>, <span class="refAuthor">Schiller, J.</span>, and <span class="refAuthor">S. Crocker</span>, <span class="refTitle">"Randomness Requirements for Security"</span>, <span class="seriesInfo">BCP 106</span>, <span class="seriesInfo">RFC 4086</span>, <span class="seriesInfo">DOI 10.17487/RFC4086</span>, June 2005, \<<https://www.rfc-editor.org/rfc/rfc4086>\>.

\[RFC7515\]  
<span class="refAuthor">Jones, M.</span>, <span class="refAuthor">Bradley, J.</span>, and <span class="refAuthor">N. Sakimura</span>, <span class="refTitle">"JSON Web Signature (JWS)"</span>, <span class="seriesInfo">RFC 7515</span>, <span class="seriesInfo">DOI 10.17487/RFC7515</span>, May 2015, \<<https://www.rfc-editor.org/rfc/rfc7515>\>.

\[RFC7516\]  
<span class="refAuthor">Jones, M.</span> and <span class="refAuthor">J. Hildebrand</span>, <span class="refTitle">"JSON Web Encryption (JWE)"</span>, <span class="seriesInfo">RFC 7516</span>, <span class="seriesInfo">DOI 10.17487/RFC7516</span>, May 2015, \<<https://www.rfc-editor.org/rfc/rfc7516>\>.

\[RFC7517\]  
<span class="refAuthor">Jones, M.</span>, <span class="refTitle">"JSON Web Key (JWK)"</span>, <span class="seriesInfo">RFC 7517</span>, <span class="seriesInfo">DOI 10.17487/RFC7517</span>, May 2015, \<<https://www.rfc-editor.org/rfc/rfc7517>\>.

\[RFC7518\]  
<span class="refAuthor">Jones, M.</span>, <span class="refTitle">"JSON Web Algorithms (JWA)"</span>, <span class="seriesInfo">RFC 7518</span>, <span class="seriesInfo">DOI 10.17487/RFC7518</span>, May 2015, \<<https://www.rfc-editor.org/rfc/rfc7518>\>.

\[RFC8174\]  
<span class="refAuthor">Leiba, B.</span>, <span class="refTitle">"Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words"</span>, <span class="seriesInfo">BCP 14</span>, <span class="seriesInfo">RFC 8174</span>, <span class="seriesInfo">DOI 10.17487/RFC8174</span>, May 2017, \<<https://www.rfc-editor.org/rfc/rfc8174>\>.

\[RFC9901\]  
<span class="refAuthor">Fett, D.</span>, <span class="refAuthor">Yasuda, K.</span>, and <span class="refAuthor">B. Campbell</span>, <span class="refTitle">"Selective Disclosure for JSON Web Tokens"</span>, <span class="seriesInfo">RFC 9901</span>, <span class="seriesInfo">DOI 10.17487/RFC9901</span>, November 2025, \<<https://www.rfc-editor.org/rfc/rfc9901>\>.

</div>

</div>

<div id="sec-informative-references">

<div id="section-10.2" class="section">

### <a href="#section-10.2" class="section-number selfRef">10.2.</a> <a href="#name-informative-references" class="section-name selfRef">Informative References</a>

\[ISO.18013-5\]  
<span class="refAuthor">ISO/IEC</span>, <span class="refTitle">"Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL) application"</span>, <span class="seriesInfo">ISO/IEC 18013-5:2021</span>, September 2021.

\[OpenID4VCI\]  
<span class="refAuthor">Lodderstedt, T.</span>, <span class="refAuthor">Yasuda, K.</span>, and <span class="refAuthor">T. Looker</span>, <span class="refTitle">"OpenID for Verifiable Credential Issuance"</span>, 2025, \<<https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html>\>.

\[OpenID4VP\]  
<span class="refAuthor">Terbu, O.</span>, <span class="refAuthor">Lodderstedt, T.</span>, <span class="refAuthor">Yasuda, K.</span>, and <span class="refAuthor">T. Looker</span>, <span class="refTitle">"OpenID for Verifiable Presentations"</span>, 2025, \<<https://openid.net/specs/openid-4-verifiable-presentations-1_0.html>\>.

\[RFC6750\]  
<span class="refAuthor">Jones, M.</span> and <span class="refAuthor">D. Hardt</span>, <span class="refTitle">"The OAuth 2.0 Authorization Framework: Bearer Token Usage"</span>, <span class="seriesInfo">RFC 6750</span>, <span class="seriesInfo">DOI 10.17487/RFC6750</span>, October 2012, \<<https://www.rfc-editor.org/rfc/rfc6750>\>.

\[RFC8414\]  
<span class="refAuthor">Jones, M.</span>, <span class="refAuthor">Sakimura, N.</span>, and <span class="refAuthor">J. Bradley</span>, <span class="refTitle">"OAuth 2.0 Authorization Server Metadata"</span>, <span class="seriesInfo">RFC 8414</span>, <span class="seriesInfo">DOI 10.17487/RFC8414</span>, June 2018, \<<https://www.rfc-editor.org/rfc/rfc8414>\>.

\[RFC8705\]  
<span class="refAuthor">Campbell, B.</span>, <span class="refAuthor">Bradley, J.</span>, <span class="refAuthor">Sakimura, N.</span>, and <span class="refAuthor">T. Lodderstedt</span>, <span class="refTitle">"OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens"</span>, <span class="seriesInfo">RFC 8705</span>, <span class="seriesInfo">DOI 10.17487/RFC8705</span>, February 2020, \<<https://www.rfc-editor.org/rfc/rfc8705>\>.

\[RFC9180\]  
<span class="refAuthor">Barnes, R.</span>, <span class="refAuthor">Bhargavan, K.</span>, <span class="refAuthor">Lipp, B.</span>, and <span class="refAuthor">C. Wood</span>, <span class="refTitle">"Hybrid Public Key Encryption"</span>, <span class="seriesInfo">RFC 9180</span>, <span class="seriesInfo">DOI 10.17487/RFC9180</span>, February 2022, \<<https://www.rfc-editor.org/rfc/rfc9180>\>.

\[RFC9458\]  
<span class="refAuthor">Thomson, M.</span> and <span class="refAuthor">C. A. Wood</span>, <span class="refTitle">"Oblivious HTTP"</span>, <span class="seriesInfo">RFC 9458</span>, <span class="seriesInfo">DOI 10.17487/RFC9458</span>, January 2024, \<<https://www.rfc-editor.org/rfc/rfc9458>\>.

\[SD-JWT-VC\]  
<span class="refAuthor">Terbu, O.</span>, <span class="refAuthor">Fett, D.</span>, and <span class="refAuthor">B. Campbell</span>, <span class="refTitle">"SD-JWT-based Verifiable Credentials (SD-JWT VC)"</span>, 2025, \<<https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/>\>.

\[W3C.DC-API\]  
<span class="refAuthor">Caceres, M.</span>, <span class="refAuthor">Yasuda, K.</span>, and <span class="refAuthor">S. Goto</span>, <span class="refTitle">"Digital Credentials"</span>, 2025, \<<https://wicg.github.io/digital-credentials/>\>.

</div>

</div>

</div>

</div>

<div id="authors-addresses">

<div id="appendix-A" class="section">

## <a href="#name-authors-address" class="section-name selfRef">Author's Address</a>

<div class="left" dir="auto">

<span class="fn nameRole">Lee Campbell</span>

</div>

<div class="left" dir="auto">

<span class="org">Google</span>

</div>

<div class="email">

Email: <leecam@google.com>

</div>

</div>

</div>
