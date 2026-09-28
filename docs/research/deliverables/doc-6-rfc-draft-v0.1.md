% PassSign: A Passkey-Native Protocol for Trusted Electronic Signatures
% Draft v0.1
% 2026-07-17

# PassSign: A Passkey-Native Protocol for Trusted Electronic Signatures

**Document 6 — Draft Community Group Report, Version 0.1**
**Status:** Published as a Draft Community Group Report of the Passkey Document Signing Community Group. Select components (CMS/JAdES attribute definitions, §9.3) are additionally intended as IETF individual Internet-Drafts. This document is written in Internet-Draft-adjacent conventions (RFC 2119/8174 keywords, Security/Privacy/IANA Considerations) to ease any future transition to the W3C Standards Track or IETF.
**Author's Note:** This draft formalizes Document 5 (Protocol Proposal) and rests on the research and gate decisions recorded in Documents 1–5 of the PassSign Research Program. It defines a protocol; it does not itself constitute legal advice, and eIDAS/ESIGN/UETA tier mappings described herein require jurisdiction-specific counsel review before reliance.

---

## Abstract

PassSign defines a signing ceremony and evidence format that composes WebAuthn passkey authorization, verifiable digital credentials, and existing digital-signature container standards (CAdES/PAdES/JAdES) into an open, interoperable protocol for legally meaningful electronic signatures. PassSign does not modify WebAuthn, does not replace eIDAS/ESIGN/UETA, and does not define new cryptographic primitives. It defines: (1) a Signing Manifest format that an application counter-signs to commit to what a signer is shown; (2) a ceremony binding a WebAuthn passkey assertion to that manifest as an authorization event; (3) an Evidence Record format, independently verifiable offline by any third party, that binds the authorization event, the resulting standard-format signature, and an identity-assurance context into one auditable artifact; and (4) conformance classes and assurance tiers that make every claim about a signature's strength machine-checkable rather than asserted.

## Status of This Document

This report was published by the [Passkey Document Signing Community Group](https://www.w3.org/community/passkey-signing/). It is not a W3C Standard nor is it on the W3C Standards Track. Please note that under the [W3C Community Contributor License Agreement (CLA)](https://www.w3.org/community/about/process/cla/) there is a limited opt-out and other conditions apply. Learn more about [W3C Community and Business Groups](https://www.w3.org/community/).

## Copyright Notice

Copyright © 2026 the Contributors to the "PassSign: A Passkey-Native Protocol for Trusted Electronic Signatures, Version 0.1" Specification, published by the [Passkey Document Signing Community Group](https://www.w3.org/community/passkey-signing/) under the [W3C Community Contributor License Agreement (CLA)](https://www.w3.org/community/about/process/cla/). A human-readable [summary](https://www.w3.org/community/about/process/cla-deed/) is available.

---

## 1. Introduction

### 1.1 Problem statement

Passkeys (WebAuthn credentials with user verification) are deployed on billions of devices for authentication. Electronic document signing remains dominated by platforms that bind signer identity to control of an email inbox by default, meter stronger identity verification per transaction, and produce audit trails whose evidentiary value depends on trusting the platform's continued existence and honesty (Document 1). Independently, WebAuthn as deployed cannot sign documents: an assertion signs `authenticatorData || SHA-256(clientDataJSON)`, never RP-chosen bytes directly, and no deployed signature container (CMS, JWS, mdoc SessionTranscript) accepts that structure as a signature value (Document 2, Part I §2, Part III §4).

The PassSign Research Program (Documents 1–4) established that closing this gap requires no new cryptography: every needed component — WebAuthn, OpenID4VP/Digital Credentials API presentations, CSC API remote signing, ETSI AdES containers, RFC 3161 timestamping — already exists, is final or Recommendation-status, and is unclaimed as a unified composition (Document 4, verdict). This document specifies that composition.

### 1.2 Scope

This document specifies: the Signing Manifest and Evidence Record formats (§6); the Enrollment, Signing, Verification, Re-binding, and Multi-Signer Composition ceremonies (§7); three conformance classes and three assurance tiers (§8); security and privacy considerations (§9–10); and an IANA-considerations-style registration plan (§11). It does not specify: workflow orchestration, routing, or notification (application layer, per Document 4 Ch. 12 and Document 5 §5.5, Decision D4); a transparency log protocol (named as an optional extension slot, §7.6, per Decision D5); or the WebAuthn `sign` extension itself (external work, tracked per §12).

### 1.3 Relationship to existing standards

PassSign is a **profile and composition**, not a replacement:

* **WebAuthn** [WEBAUTHN-3] is used unmodified; PassSign defines how its assertion output is bound to a manifest, not new authenticator behavior. Class C (§8.1) anticipates the WebAuthn `sign` extension [WEBAUTHN-SIGN-PR] without depending on it.
* **OpenID4VP / Digital Credentials API** [OID4VP] [DC-API] are used unmodified for identity presentation at enrollment and re-binding.
* **CMS, CAdES, PAdES, JAdES** [RFC5652] [CAdES] [PAdES] [JAdES] are the baseline signature containers; PassSign's Class B (§8.1) produces byte-conformant output.
* **CSC API** [CSC-API] is the profiled interface between the Orchestrating Relying Party and the Signing Service (§9.3).
* **RFC 3161** [RFC3161] timestamps every Evidence Record unmodified.
* **eIDAS, ESIGN, UETA** [eIDAS] [ESIGN] [UETA] are the legal frameworks PassSign's assurance tiers are designed to map onto (§8.2), per Document 3's analysis; this document makes no independent legal claim.

## 2. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals as shown here.

**Signer**: A natural person holding a WebAuthn passkey, optionally holding a Digital Identity Credential.

**Digital Identity Credential**: A W3C Verifiable Credential, SD-JWT VC, or ISO/IEC 18013-5 mdoc presented by the Signer at Enrollment or Re-binding, used to establish the identity bound to the Signer's PassSign account.

**Orchestrating Relying Party (ORP)**: The application initiating a signing ceremony and constructing the Signing Manifest.

**Signing Service (SS)**: The party (which MAY be the ORP; §8.2) custodying signing key material and performing the Activation Check (§7.2).

**Signing Manifest**: A JSON object, counter-signed by the ORP, that commits the ORP to a claimed document hash set and a claimed human-readable representation of the signing transaction (§6.1).

**Activation Assertion**: A WebAuthn assertion whose challenge is bound to the hash of a specific Signing Manifest.

**Activation Check**: The SS's verification, prior to producing a signature, that a valid Activation Assertion exists for the exact Signing Manifest presented.

**Evidence Record**: A JSON object, signed by the SS, that is the complete, independently verifiable record of one signing event (§6.2).

**Signer Receipt**: A copy of the complete Evidence Record delivered to the Signer's agent at ceremony completion (§7.2 step 6; normative, Decision D5).

**Conformance Class**: One of Class A, B, or C (§8.1), describing which cryptographic mechanism produced the signature.

**Assurance Tier**: One of Tier 1, 2, or 3 (§8.2), describing the strength of authenticator, identity binding, and Signing Service operation behind a given Evidence Record.

**Verifier**: Any party validating a signed document and its Evidence Record, per the normative procedure in §7.3.

## 3. Actors and Roles

(Normative summary; full architecture rationale in Document 5 §2–§3.)

```
   Identity Issuer -----(enrollment credential)----> Signer
   Signer <--(manifest, ceremony)--> ORP
   Signer --(Activation Assertion)--> SS
   SS --(signed doc + Evidence Record)--> ORP
   SS --(Signer Receipt)--> Signer
   SS --(timestamp request)--> TSA
   [OPTIONAL] SS --(record hash)--> Transparency Log
   Verifier: consumes {signed doc, Evidence Record} independently of all of the above
```

An ORP and an SS MAY be operated by the same entity (Tier 1 only; §8.2). PassSign conformance requirements bind to the **role**, never to the operating entity.

## 4. Design Requirements (Normative Goals)

A conforming implementation MUST satisfy:

* **REQ-1**: Any Evidence Record MUST be verifiable by a Verifier possessing only the Evidence Record, the signed document, and public key material (SS public keys, Identity Issuer trust anchors, TSA certificates) — no query to the ORP, SS, or Identity Issuer at verification time (offline verification).
* **REQ-2**: An Evidence Record MUST NOT be produced unless a fresh Activation Assertion exists whose challenge is cryptographically bound to the exact Signing Manifest hash presented for that signature.
* **REQ-3**: At Conformance Class B, the signed document MUST validate as a standards-conformant PAdES, CAdES, or JAdES signature in existing, unmodified validation software.
* **REQ-4**: The Signing Service role MUST be substitutable: any conforming SS implementation MUST be usable by any conforming ORP via the interface in §9.3, and no PassSign conformance requirement MAY depend on a single named operator.
* **REQ-5**: Weak-factor credentials (e.g., email OTP, SMS OTP, static recovery codes) MUST NOT be accepted as sufficient authentication for the Re-binding ceremony (§7.4).

## 5. Conventions: Objects as JSON

All PassSign objects are UTF-8 JSON, signed as Compact Serialization JWS [RFC7515] unless otherwise stated. Field names use `snake_case`. Binary fields are Base64URL-encoded without padding, per WebAuthn convention [WEBAUTHN-3] §5.8.1. This document defines objects by annotated field table rather than formal JSON Schema; a normative JSON Schema and IANA media-type registration are open items (§11, §13).

## 6. Object Definitions

### 6.1 Signing Manifest

A JSON object, signed by the ORP as a JWS (`typ: "passsign-manifest+jwt"`, proposed media type).

| Field | Type | Requirement | Description |
|---|---|---|---|
| `doc` | array of objects | REQUIRED | One entry per document state to be signed: `{alg, hash, byte_range?}`. `alg` MUST be a registered hash algorithm identifier (e.g., `sha-256`). |
| `display` | object | REQUIRED | Human-readable transaction representation shown to the Signer: `title`, `parties` (array), `summary`, `consent_text`, `locale`. This is the ORP's committed claim of what was rendered (§7.1 step 1; §9.2). |
| `tier` | string | REQUIRED | Claimed Assurance Tier: `"1"`, `"2"`, or `"3"` (§8.2). |
| `class` | string | REQUIRED | Claimed Conformance Class: `"A"`, `"B"`, or `"C"` (§8.1). |
| `orp` | object | REQUIRED | ORP identifier and key reference for counter-signature verification. |
| `signer_ref` | string | REQUIRED | SS-scoped pseudonymous account reference for the Signer. |
| `order` | object | OPTIONAL | Declared multi-signer position, e.g. `{"position": 2, "of": 3}`. Recorded only; not enforced by this protocol (§7.5, Decision D4). |
| `policy` | object | OPTIONAL | Timestamping policy, transparency-log usage flag (§7.6). |
| `nonce` | string | REQUIRED | Fresh random value, replay protection. |
| `iat` | integer | REQUIRED | Issued-at time (Unix seconds). |

The Manifest JWS's signature by the ORP is the **ORP counter-signature** (Decision D5; normative, §9.2).

### 6.2 Evidence Record

A JSON object, signed by the SS as a JWS (`typ: "passsign-evidence+jwt"`, proposed media type).

| Field group | Contents |
|---|---|
| `manifest` | The complete Signing Manifest JWS from §6.1 (embeds the ORP counter-signature). |
| `assertion` | `authenticator_data`, `client_data_json` (verbatim, Base64URL), `signature`, `credential_id`, credential public key (COSE format), `authenticator_class` (`"synced"` \| `"device-bound"`), `attestation` (present/absent, format if present), `uv` (boolean, MUST be `true`). |
| `identity` | `issuer`, `credential_type`, `trust_anchor_ref`, `assurance_context` (e.g. eIDAS LoA or NIST IAL, if declared by the issuer), disclosed-claims subset, `status_at_enrollment`, `status_at_signing` (§8.3 snapshot rule). |
| `binding` | `chain_head` (hash of the prior event in the account's evidence chain), `event_type` (`"enrollment"` \| `"signing"` \| `"rebinding"`), and for `"rebinding"` events, a reference to the superseded credential (Decision D3 visibility requirement). |
| `container` | (Class B/C only) reference to the produced standard signature: format (`PAdES`/`CAdES`/`JAdES`), `byte_range_digest`, `certificate_chain_ref`. Absent at Class A. |
| `sig_service` | SS identifier, key reference, declared Tier, and a machine-readable **activation-check statement** attesting the check in §7.2 step 4 was performed. |
| `timestamps` | One or more RFC 3161 [RFC3161] tokens over the Evidence Record hash. |
| `log` | OPTIONAL. Transparency-log inclusion proof (§7.6). |

## 7. Ceremonies

### 7.1 Enrollment

1. The Signer presents a Digital Identity Credential to the SS via OpenID4VP or the Digital Credentials API [OID4VP] [DC-API].
2. The SS MUST validate the presentation against its configured trust anchors and MUST record `issuer`, `credential_type`, `trust_anchor_ref`, and `assurance_context`.
3. The Signer registers a WebAuthn credential at the SS's RP ID with `userVerification: "required"`. The SS MUST record whether attestation was obtained and, if so, its format.
4. (Class B) The SS MUST provision a key pair in a hardware security module and MUST obtain an X.509 certificate for it. One-time, per-ceremony certificates MAY be used [ETSI-119431-1].
5. The SS MUST open an **account evidence chain**: an append-only, hash-linked sequence of events beginning with this enrollment event.

### 7.2 Signing

1. The ORP MUST construct a Signing Manifest (§6.1) reflecting exactly what will be presented to the Signer, and MUST sign it (ORP counter-signature).
2. The ORP submits the Manifest to the SS via the interface in §9.3.
3. The SS MUST validate the Manifest schema and the ORP counter-signature, then MUST issue a WebAuthn `get()` call with `challenge = BASE64URL(SHA-256(manifest_jws))` and `userVerification: "required"`.
4. The Signer completes user verification. The SS MUST perform the **Activation Check**: the returned assertion verifies against the Signer's enrolled credential public key; the challenge equals the Manifest hash computed in step 3; the `uv` flag is set; the account is active; and Tier constraints (§8.2) are met.
5. **Class B**: the SS's HSM produces a standard container signature over the document(s) referenced in `doc`. **Class A**: the Activation Assertion is itself the terminal cryptographic artifact; no container signature is produced.
6. The SS MUST assemble the Evidence Record (§6.2), obtain at least one RFC 3161 timestamp over it, append the signing event to the account evidence chain, deliver the signed document and Evidence Record to the ORP, and deliver the **Signer Receipt** (a complete copy of the Evidence Record) to the Signer. If transparency-log policy is enabled (§7.6), the SS MUST submit the record hash to the configured log.

### 7.3 Verification

A conforming Verifier, given a signed document and an Evidence Record, MUST perform, in order, and MUST treat failure of any step as overall verification failure:

1. Validate the Evidence Record JWS signature against the SS's public key.
2. Validate the ORP counter-signature on the embedded Manifest.
3. Validate the WebAuthn assertion in `assertion` against the credential public key on file for the account (obtained out-of-band or embedded per deployment policy), including recomputing `challenge` from the embedded Manifest and confirming equality, and confirming `client_data_json.type == "webauthn.get"`.
4. (Class B/C) Confirm the document hash(es) in `manifest.doc` equal the actual signed document's digest(s), and confirm `container.byte_range_digest` is consistent with the document's embedded signature.
5. Validate all timestamp tokens in `timestamps`.
6. Validate `identity.status_at_signing` against the snapshot rule (§8.3) — no live query required.
7. Confirm internal consistency: the declared `tier` and `class` in the Manifest are consistent with the `authenticator_class`, `attestation`, and `sig_service` fields actually present (e.g., a Tier 2 claim accompanied by `authenticator_class: "synced"` and no attestation MUST fail verification).

A Verifier MUST NOT report a PassSign signature as valid on the basis of standard container validation (e.g., "Adobe shows a green checkmark") alone; per REQ-2 and Decision D2, container validity without Evidence Record validity is not a PassSign signature.

### 7.4 Re-binding

Triggered when a Signer presents a new WebAuthn credential for an existing account.

1. If the presented credential is the same credential synced to a new device, this is not Re-binding; no special ceremony applies.
2. Otherwise, the SS MUST require either: (a) a fresh Digital Identity Credential presentation whose disclosed attributes are consistent, per SS policy, with the identity recorded at Enrollment; or (b) a **cross-signing** ceremony in which the still-available old credential authorizes registration of the new one.
3. Weak-factor credentials (email OTP, SMS OTP, static recovery codes) MUST NOT satisfy step 2 (REQ-5; Decision D3).
4. The SS MUST append a `"rebinding"` event to the account evidence chain, referencing the superseded credential. This event MUST be discoverable in the `binding` field of every subsequently produced Evidence Record for the account (Decision D3).
5. An account with no Digital Identity Credential eligible for re-presentation and no available prior credential for cross-signing cannot Re-bind; such a Signer MUST enroll a new account. Evidence Records already produced under the old account are unaffected.

### 7.5 Multi-Signer Composition

1. Each Signer's ceremony (§7.2) binds to the document state that Signer actually observed and signed; `manifest.doc` MUST reference that exact state (e.g., via incremental-update byte ranges for PDF).
2. A later Signer's signature, by covering the then-current document state, transitively covers earlier signatures. Verifiers MAY use per-signature timestamps and document-state coverage to establish signing order after the fact.
3. `manifest.order` MAY declare an intended sequence position; this protocol does NOT enforce sequencing. Enforcement of "Signer B may not sign before Signer A," routing, reminders, and expiry are OUT OF SCOPE and are an application-layer (ORP) concern (Decision D4; Document 4 Ch. 12).

### 7.6 Transparency Log (Optional Extension)

An SS MAY, per deployment policy, submit each Evidence Record's hash to an append-only transparency log and embed the resulting inclusion proof in the `log` field. This document defines the extension **slot** (the `log` field and its semantics) but does not mandate an operator, log format, or protocol; deployments requiring this property (e.g., regulated ecosystems) MAY mandate it by local policy (Decision D5). Specification of the log protocol itself is deferred (§12).

## 8. Conformance Classes and Assurance Tiers

### 8.1 Conformance Classes

* **Class A — Assertion Evidence.** No SS-held signing key. The Activation Assertion itself, bound to the Manifest hash, is the terminal artifact. No standard signature container is produced; a Verifier relies solely on Evidence Record validation. Implementations MUST use a credential dedicated to signing (not shared with login) to avoid key entanglement (Document 2, Part I, Limitation L-6).
* **Class B — Authorized Signing Key.** The Activation Assertion authorizes an SS-held HSM key to produce a standard PAdES/CAdES/JAdES container signature (§7.2 step 5). This is the RECOMMENDED baseline class for v0.1 implementations (REQ-3).
* **Class C — Native Extension Signing.** Anticipates the WebAuthn `sign` extension [WEBAUTHN-SIGN-PR]: a derived, attestable key signs the container's signing input directly. NOT RECOMMENDED for production use before the referenced extension reaches broad implementation (tracked in §12); this document reserves the class identifier and Evidence Record shape for forward compatibility.

### 8.2 Assurance Tiers

| Tier | Authenticator requirement | SS requirement | Identity requirement | Indicative legal posture (Document 3; not legal advice) |
|---|---|---|---|---|
| **1 — Baseline** | Any conforming passkey, including synced | ORP and SS MAY be the same entity | Any deployment-policy-accepted identity check, labeled | SES-equivalent with strong attribution evidence |
| **2 — Assured** | Device-bound, attestation present | Independent from ORP, or SS runs a certified activation module analogous to [EN419241-2] SCAL2 | Cryptographically-bound Digital Identity Credential at substantial-or-higher assurance | AdES-equivalent, per Art. 26 mapping analysis in Document 3 §2 |
| **3 — Qualified** | Tier 2 requirements | QTSP-operated per [CSC-API]/[ETSI-119432], certified QSCD | LoA-high-equivalent Digital Identity Credential | QES-equivalent, reached via QTSP qualification, not by this protocol alone |

A Tier claim in a Manifest MUST be checkable against the corresponding fields of the resulting Evidence Record (§7.3 step 7); this protocol does not permit an unchecked, asserted Tier label.

### 8.3 Status Snapshot Rule

All revocation/status information (identity-credential status-list proofs, certificate OCSP/CRL responses) MUST be evaluated at signing time and embedded in the Evidence Record's `identity.status_at_signing` and `container` fields. A Verifier MUST NOT be required to make a live query to any issuer or the SS to complete verification (REQ-1).

## 9. Security Considerations

This section elaborates the threat analysis in Document 5 §9.

### 9.1 Forged signatures via a compromised or malicious Signing Service

At Tier 1, where ORP and SS may be a single entity, that entity could attempt to produce a container signature without a genuine Activation Assertion. Per REQ-2 and the verification procedure in §7.3, such an artifact would fail Evidence Record validation even though the underlying container signature might independently validate in unmodified signature tooling. Implementers and downstream Verifiers MUST treat container-only validation as insufficient (§7.3, closing paragraph). At Tier 2+, this risk is further reduced by requiring an independent SS or certified activation module.

### 9.2 Manifest misrepresentation ("bait-and-switch")

An ORP could display content to the Signer inconsistent with the `display` field of the Manifest it constructs, or could construct a Manifest whose `doc` hashes do not correspond to the displayed document. The ORP counter-signature (§6.1) makes the ORP's claim about what was shown cryptographically attributable and permanent; a mismatch between the counter-signed `display` claim and the actual content of the hashed document is detectable by any party who later compares them, though not preventable at ceremony time (no trusted-display mechanism is assumed or required by this protocol; this is a known, industry-wide limitation — Document 2, Part I §4.2, on the withdrawal of the CTAP2.0 transaction-confirmation extensions). Implementers SHOULD retain the Signer Receipt as the Signer's independent evidence of the ORP's claim.

### 9.3 Evidence suppression and history control

An ORP or SS could attempt to suppress or alter an Evidence Record after the fact. The Signer Receipt (§7.2 step 6, normative) ensures the Signer independently holds an unmodifiable copy. The account evidence chain's hash-linking (§7.1 step 5, §7.2 step 6) makes deletion or reordering of an account's history detectable by comparing chain heads across copies held by different parties. Deployments with elevated requirements SHOULD enable the transparency-log extension (§7.6).

### 9.4 Replay and cross-ceremony reuse

The Activation Assertion's challenge is bound to a specific Manifest's hash, which includes a fresh nonce and issued-at time (§6.1); an assertion cannot be replayed against a different Manifest, and a Manifest cannot be reused to request a second ceremony.

### 9.5 Sole control and key custody

At Class B/Tier 2+, "sole control" of the signing key (relevant to AdES-equivalent claims, Document 3 §2) is satisfied by the Signer's exclusive capability to produce a valid Activation Assertion, not by physical custody of the HSM-held key — analogous to the accepted remote-QSCD/SCAL2 pattern [EN419241-2]. This document does not itself certify SS implementations against that pattern; deployments claiming Tier 2/3 status SHOULD pursue applicable certification.

### 9.6 Recovery and re-binding attacks

Per REQ-5 and §7.4, weak-factor recovery paths are excluded from Re-binding by design, closing the class of attack (SIM-swap-style, inbox-compromise-style) that undermines many authentication systems' recovery flows. The residual risk — compromise of the Signer's Digital Identity Credential itself, or coercion — is inherited from the identity-issuance ecosystem and is out of scope for this document.

### 9.7 Synced-credential key exposure

Synced passkeys' private key material is replicated via platform provider infrastructure (Document 2, Part I §5) without authenticator attestation. This document addresses the resulting risk structurally, via the Tier system (§8.2), rather than by prohibiting synced credentials: Tier 1 accepts them with a correspondingly limited legal-posture claim; Tier 2+ requires device-bound, attested credentials.

## 10. Privacy Considerations

Evidence Records contain identity-credential disclosures (§6.2 `identity`), which SHOULD be limited to the minimum claims necessary for the ORP's stated purpose, consistent with the selective-disclosure mechanisms of the underlying credential formats (SD-JWT, mdoc; Document 2 Part II §4). WebAuthn credentials are scoped per-SS RP ID by underlying platform behavior, limiting cross-context correlation by the SS itself; ORPs and any transparency-log operator SHOULD avoid introducing correlation handles not already present in the credential presentation. A full data-minimization and applicable-law (e.g., GDPR) review is an open item for the next revision of this document (§13, item 8) and is a prerequisite the authors consider necessary before any production deployment, not merely a documentation nicety.

## 11. IANA-Style Considerations

This section records registrations this specification would require if progressed through IETF or an equivalent registry-holding body; no registration has yet been requested.

* Media type `application/passsign-manifest+jwt` for the Signing Manifest (§6.1).
* Media type `application/passsign-evidence+jwt` for the Evidence Record (§6.2).
* A PAdES SubFilter value and/or CMS `signatureAlgorithm` OID for Class A/assertion-format evidence, if and when the container-mapping profile referenced in Document 4 Ch. 4 is specified (deferred; see §13 item 3).
* A JOSE/COSE algorithm identifier for WebAuthn-assertion-shaped signatures, under the same deferral.

## 12. Relationship to In-Flight External Work

This document's Class C (§8.1) and its long-term interoperability depend in part on external, not-yet-stable work:

* **w3c/webauthn PR #2078** ("Add 'sign' extension") [WEBAUTHN-SIGN-PR]: the PassSign project has submitted public comment on this PR identifying document-signing as a distinct use case from the wallet-holder-binding cases discussed there, and raising three requirements: registered algorithm identification compatible with CMS/JOSE/COSE containers, RP-discoverable attestation status at signing time, and an explicit domain-separation convention for the signed input. This document's Class C is deliberately underspecified pending that discussion's outcome.
* **WebKit `remote-cryptokeys`**: a possibly-overlapping proposal; convergence with the above is, as of this draft, an open question raised in the same PR thread.
* **ETSI TS 119 432 / CSC API evolution**: §9.3 (deferred detailed profile) tracks the current versions cited in §13; future revisions of those specifications may require corresponding updates here.

## 13. Open Items for the Next Revision

Carried forward from Document 5 §11, restated as concrete drafting tasks:

1. Formal JSON Schema for the Signing Manifest and Evidence Record; IANA media-type registration request.
2. JAdES mapping annex: a field-by-field, lossless mapping from the Evidence Record's JSON structure to JAdES `etsiU` unsigned-attribute carriage, with a proof obligation that no information is lost in either direction (Decision D1).
3. A concrete CSC API v2.x [CSC-API] profile for the ORP–SS interface referenced in §7.2, including the manifest-submission and Evidence-Record-retrieval extensions to that API.
4. Cross-origin ceremony embedding specification (permission policy for `publickey-credentials-get` when ORP ≠ SS's RP ID; redirect-based fallback; required user-facing disclosure text).
5. Formal account evidence chain data structure (hash-link algorithm, pruning and archival rules for long-lived accounts).
6. Transparency log extension specification (§7.6): entry format, inclusion-proof carriage, and an operator-neutral log-discovery mechanism.
7. Internationalization requirements for the `display` field and accessibility requirements for the signing ceremony's user interface.
8. Formal privacy/data-minimization review (§10), including applicable-law analysis, prior to any production reference implementation.
9. A complete Class A container-less verification profile, and finalization of Class C's Evidence Record shape once [WEBAUTHN-SIGN-PR] stabilizes.
10. A conformance test-vector suite sufficient for an independent implementer to build a verifier from this document and the test vectors alone, without reference to any reference implementation's source code (REQ-1, design goal G4 of Document 5).

## 14. Acknowledgments

This document rests on the PassSign Research Program (Documents 1–5), which drew on the published work of the W3C WebAuthn, Federated Identity, and Web Payments Working Groups; the FIDO Alliance; the IETF COSE, JOSE, LAMPS, and OAuth working groups; ETSI TC ESI; the OpenID Foundation Digital Credentials Protocols Working Group; the Cloud Signature Consortium; and the EU Digital Identity Wallet Architecture and Reference Framework contributors. Specific PR discussion referenced in §12 is the work of its named authors on w3c/webauthn.

## 15. References

### 15.1 Normative References

* [RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119.
* [RFC8174] Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174.
* [RFC7515] Jones, M., Bradley, J., Sakimura, N., "JSON Web Signature (JWS)", RFC 7515.
* [RFC5652] Housley, R., "Cryptographic Message Syntax (CMS)", RFC 5652.
* [RFC3161] Adams, C., et al., "Internet X.509 Public Key Infrastructure Time-Stamp Protocol (TSP)", RFC 3161.
* [WEBAUTHN-3] W3C, "Web Authentication: An API for accessing Public Key Credentials - Level 3".
* [OID4VP] OpenID Foundation, "OpenID for Verifiable Presentations 1.0".
* [DC-API] W3C Federated Identity Working Group, "Digital Credentials API".
* [CAdES] ETSI EN 319 122, "CAdES digital signatures".
* [PAdES] ETSI EN 319 142, "PAdES digital signatures".
* [JAdES] ETSI TS 119 182, "JAdES digital signatures".
* [CSC-API] Cloud Signature Consortium, "Cloud Signature Consortium API", v2.2.
* [ETSI-119432] ETSI TS 119 432, "Protocols for remote digital signature creation".
* [ETSI-119431-1] ETSI TS 119 431-1, "Policy and security requirements for remote signature creation".
* [EN419241-2] CEN EN 419 241-2, "Security Requirements for Trustworthy Systems Supporting Server Signing".

### 15.2 Informative References

* [WEBAUTHN-SIGN-PR] w3c/webauthn PR #2078, "Add 'sign' extension".
* [eIDAS] Regulation (EU) No 910/2014, as amended by Regulation (EU) 2024/1183.
* [ESIGN] 15 U.S.C. § 7001 et seq.
* [UETA] Uniform Electronic Transactions Act (1999).
* PassSign Research Program, Documents 1–5 (this project, `docs/research/`).

---

*This document describes a protocol design and cites legal frameworks for architectural purposes only. Nothing in this document constitutes legal advice; jurisdiction-specific counsel review is required before any assurance-tier claim is relied upon in production.*
