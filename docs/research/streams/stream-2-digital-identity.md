# Stream 2: Digital Identity

**PassSign research program — identity independent from authentication**
Date of research: 2026-07-14. Author: Stream 2 research agent.

> **Method note.** The WebSearch tool was not available in this environment (ToolSearch returned no match), and the workspace shell has no outbound network. All research was therefore performed with direct fetches (`web_fetch`) of primary sources. The fetch tool truncates pages at roughly 95–115 KB, so for very long specifications (VC-DM 2.0, OpenID4VCI, OpenID4VP, ARF main document, NIST SP 800-63A) only the first portion of the document plus its complete table of contents was readable; appendix-level details of OpenID4VCI (Appendices D/E/F) were confirmed via the table of contents of the spec itself and via the normative cross-references in the OpenID4VC High Assurance Interoperability Profile (HAIP), which was fetched separately. One source failed outright: `https://fidoalliance.org/digital-credentials/` returned an empty page body; claims about FIDO Alliance's digital-credentials work are therefore omitted rather than sourced second-hand. These limitations are flagged inline where they matter.

---

## 1. W3C Verifiable Credentials 2.0: Data Model & Proof Formats

### 1.1 Roles and data model

The W3C Verifiable Credentials Data Model 2.0 (VCDM 2.0) is a W3C Recommendation (the VC 2.0 family, including the Data Model, Bitstring Status List, Data Integrity, and VC-JOSE-COSE, was published as Recommendations on 15 May 2025 — the Bitstring Status List status line "W3C Recommendation 15 May 2025" was verified directly [8]). The model defines three core roles ([1], §Terminology):

- **Issuer** — "a role an entity can perform by asserting claims about one or more subjects, creating a verifiable credential from these claims, and transmitting the verifiable credential to a holder."
- **Holder** — "a role an entity might perform by possessing one or more verifiable credentials and generating verifiable presentations from them. A holder is often, but not always, a subject of the verifiable credentials they are holding."
- **Verifier** — receives presentations and verifies them.

A **presentation** is "data derived from one or more verifiable credentials issued by one or more issuers that is shared with a specific verifier" ([1], §Terminology). Holders keep credentials in a *credential repository* (wallet).

Crucially for PassSign, the data model itself carries **no signature semantics**: a *conforming document* "MUST be secured using a securing mechanism as described in Section 4.12 Securing Mechanisms," and conforming issuers "MUST secure the conforming documents they produce" ([1], §Conformance and §4.12).

### 1.2 Two proof families

VCDM 2.0 deliberately supports two disjoint securing approaches ([1], §§4.12, and the informative "information graph" figures):

1. **Embedded proofs (Data Integrity)** — a `proof` property inside the JSON-LD document, per *Verifiable Credential Data Integrity 1.0*. The VC 2.0 test-suite/conformance labels enumerate the cryptosuites: `ecdsa` (ecdsa-rdfc-2019), `ecdsa-sd` (ecdsa-sd-2023, selectively disclosable), `bbs` (bbs-2023, unlinkable selective disclosure), each "Secured with Data Integrity" ([1], conformance section). Data Integrity proofs require RDF canonicalization before signing.
2. **Enveloping proofs (JOSE/COSE)** — the credential is wrapped in a JWS or COSE_Sign1 envelope per *Securing Verifiable Credentials using JOSE and COSE* (VC-JOSE-COSE); the VC-DM refers to these as enveloping proofs and depicts them carried as `data:` URLs inside an EnvelopedVerifiableCredential ([1], Figures 7–8 discussion).

**Assessment:** the industry has largely converged on enveloped/JOSE-COSE-style signing (SD-JWT VC, mdoc) rather than Data Integrity, because JOSE/COSE avoids JSON-LD canonicalization complexity and aligns with existing PKI tooling. Both major wallet ecosystems surveyed below (EUDI, Apple, Google) use SD-JWT VC and/or ISO mdoc, not JSON-LD Data Integrity credentials.

### 1.3 Lifecycle: issuance, presentation, revocation

- **Issuance/presentation** are out of scope of the data model (protocol-agnostic); in practice they are performed by OpenID4VCI/OpenID4VP (Section 3) or ISO 18013-5 device retrieval.
- **Revocation/status** is modelled via the `credentialStatus` property, whose dominant concrete mechanism is **Bitstring Status List v1.0** (W3C Recommendation, 15 May 2025 [8]). The issuer publishes a status-list credential containing a GZIP-compressed bitstring (`encodedList`); each issued credential carries `statusListIndex` and `statusPurpose` (`revocation`, `suspension`, or `message`). The minimum list length is **131,072 entries**, chosen for herd privacy — "statusListIndex is the only link between" the credential and the list entry ([8], §§2–3, algorithms in §4: validation requires fetching the list, decompressing, and checking the bit at `statusListIndex`). Group privacy is imperfect: the verifier learns the issuer and list URL when dereferencing. The IETF equivalent for JOSE ecosystems is OAuth Token Status List (draft-ietf-oauth-status-list; referenced by HAIP §7 as "the status information of the Verifiable Credential or Wallet Attestation" [7]).

---

## 2. DIDs — and Adoption Reality

### 2.1 What DIDs are

*Decentralized Identifiers (DIDs) v1.0* is a W3C Recommendation (19 July 2022). "Decentralized identifiers (DIDs) are a new type of identifier that enables verifiable, decentralized digital identity. A DID refers to any subject… as determined by the controller of the DID" ([2], Abstract). A DID resolves to a **DID document** that "can express cryptographic material, verification methods, or services, which provide a set of mechanisms enabling a DID controller to prove control of the DID" ([2], §1). A **verification method** is typically a public key; proving control means demonstrating possession of the corresponding private key ([2], §Terminology, §5.2).

Main methods relevant here:

- **did:web** — DID document hosted at a well-known HTTPS path; trust anchored in DNS/TLS and the domain's reputation. This is what Microsoft Entra Verified ID uses: "Microsoft currently supports the `did:web` trust system… a permission-based model that allows trust using a web domain's existing reputation. The `did:web` method is generally available" ([9]).
- **did:key** — the DID *is* a multibase-encoded public key; the DID document is derived deterministically; no registry, no rotation. (W3C CCG specification, not a Recommendation [10].)
- **did:jwk** — same idea with a base64url-encoded JWK; useful as a thin wrapper to put an ephemeral key where a DID is syntactically required [11].

### 2.2 Adoption reality, 2025–2026

The honest picture is that **DIDs have not become the identifier layer of the mainstream digital-identity deployments**:

- **The EUDI Wallet ARF does not use DIDs.** In the fetched portion of the ARF v2.9.0 main document (~114 KB covering the introduction, roles, trust model and trust-relationship sections), there is **no occurrence of `did:` or DID-based identifiers at all**; the trust model is built entirely on **X.509 trust anchors published in Trusted Lists / Lists of Trusted Entities** ([5], §3.5, §6.2). Issuer authentication in HAIP is likewise `x5c` certificate chains, and HAIP explicitly mandates X.509: "the public key certificate… used to validate the signature on the Wallet Attestation MUST be included in the `x5c` JOSE header" ([7], §4.4.1); same pattern for key attestations ([7], §4.5.1). (Caveat: the ARF main document could not be read in full due to fetch truncation; the conclusion is corroborated by HAIP and by the ARF's Trusted-List-centric trust sections that were read.)
- **ISO mdoc ecosystems (Apple/Google state IDs)** use IACA X.509 certificate hierarchies, not DIDs ([12], [13]).
- **Microsoft Entra Verified ID** is the largest commercial VC deployment that *does* use DIDs — but it retreated from ledger-based `did:ion` to `did:web`, i.e., to ordinary domain trust ([9]).
- SD-JWT VC and OpenID4VCI/VP are **identifier-agnostic**: keys are conveyed as JWKs, `x5c` chains, or (optionally) DIDs. DIDs are an optional serialization detail, not a dependency.

**Implication for PassSign:** nothing in the modern VC stack forces a DID dependency. If PassSign needs stable signer identifiers, `did:jwk`/`did:key` are available as free-standing key wrappers, and `did:web` as an organization identifier — but X.509/`x5c` is the path of maximum interoperability with EUDI and mdoc verifiers.

---

## 3. OpenID4VC & the Digital Credentials API

### 3.1 OpenID4VCI (issuance) — Final 1.0

OpenID for Verifiable Credential Issuance is a **Final** OpenID Foundation specification. It layers issuance on OAuth 2.0 rails: authorization endpoint (or pre-authorized code grant), token endpoint, plus new endpoints ([3], §3.1):

- an optional **Nonce Endpoint** "from which a fresh `c_nonce` value can be obtained to be used in proof of possession of key material in a subsequent request to the Credential Endpoint";
- the **Credential Endpoint**, which "may bind an issued Credential to specific cryptographic key material, in which case Credential requests include proof(s) of possession or attestations for the key material. Multiple key proof types are supported."

The Credential Request carries a `proofs` object — "one or more proof of possessions of the cryptographic key material to which the issued Credential instances will be bound" ([3], §8.2). §8.1 states the normative binding posture: "The issued Credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the Credential. Cryptographic Key Binding allows the Verifier to verify during the presentation… that the End-User presenting a Credential is the same End-User to whom that Credential was issued." Proof types (Appendix F, per ToC and HAIP): **`jwt`** (holder-signed JWT PoP over `c_nonce` + issuer audience), **`di_vp`** (Data Integrity presentation), and **`attestation`** (a key attestation instead of a live PoP). Appendix D defines **Key Attestations** (JWT format, media type `application/key-attestation+jwt`); Appendix E defines **Wallet Attestations in JWT format**, used for client authentication via the `OAuth-Client-Attestation` / `OAuth-Client-Attestation-PoP` headers ([3], §15.4.4, Appendices D–F — confirmed via spec ToC and body references; full appendix text unreadable due to fetch truncation, but requirements confirmed in HAIP [7] §§4.4.1, 4.5.1).

The spec also names the alternatives to cryptographic binding: **Claims-based Holder Binding** ("proving certain claims, e.g., name and date of birth… allows long-term, cross-device use of a Credential as it does not depend on cryptographic key material stored on a certain device," e.g., a diploma) and issuance **without any binding** ([3], §2 definitions, §§14.1–14.2).

### 3.2 OpenID4VP (presentation) — Final 1.0

OpenID for Verifiable Presentations, also **Final**, "defines a mechanism on top of OAuth 2.0 for requesting and delivering Presentations of Credentials… of any format, including… W3C Verifiable Credentials Data Model, ISO mdoc, and IETF SD-JWT VC" ([4], §1). Key elements:

- **DCQL** (Digital Credentials Query Language), a JSON query language replacing Presentation Exchange; the request carries a `dcql_query` parameter ([4], §§5–6).
- **Same-device and cross-device flows** (§3.1–3.2), with `direct_post` response mode "to support sending the response across devices" (§4).
- **OpenID4VP over the Digital Credentials API** — a self-contained Appendix A profile: "the specification defines a separate mechanism where OpenID4VP messages are sent and received over the Digital Credentials API (DC API) instead of HTTPS messages and redirects" ([4], §1, §3, Appendix A). Encrypted responses use `dc_api.jwt`.
- Notably, §5.3 covers "Requesting Presentations **without** Holder Binding Proofs" — holder binding is expected but optional per ecosystem policy ([4], §5.3, §14.1.1).

### 3.3 Digital Credentials API (browser layer) — status mid-2026

The W3C **Digital Credentials API** is developed by the **Federated Identity Working Group**; the fetched Editor's/TR snapshot is a **W3C Working Draft dated 16 June 2026** on the Recommendation track ([6], Status section) — i.e., still pre-CR as of mid-2026, though shipping in Chromium and partially in Safari. Entry points are `navigator.credentials.get()` for presentation and `navigator.credentials.create()` for issuance, with a `DigitalCredential` interface, protocol-registry enumerations (`DigitalCredentialPresentationProtocol`, `DigitalCredentialIssuanceProtocol`), and feature detection via `DigitalCredential.userAgentAllowsProtocol()` ([6], §7). Protocols (e.g., OpenID4VP-over-DC-API, ISO mdoc request) are registered and adopted progressively by user agents. OpenID4VP defines the DC API as covering both the W3C browser API "and its equivalent native APIs on App Platforms (such as Credential Manager on Android)" ([4], §2). The EUDI ARF tracks adoption as Discussion "Topic F — Digital Credentials API" ([5]).

**Significance for PassSign:** the DC API gives the browser the same mediating role for identity presentation that WebAuthn gives it for authentication — the two ceremonies can be composed in a single web session on the same surface.

---

## 4. Selective Disclosure: SD-JWT and mdoc

### 4.1 SD-JWT — now RFC 9901

Selective Disclosure for JWTs was published as **RFC 9901 (Proposed Standard, November 2025)** ([14], datatracker: "RFC – Proposed Standard (November 2025), was draft-ietf-oauth-selective-disclosure-jwt"). Mechanics: the issuer replaces claim values with salted hash digests; the actual values travel as separate **Disclosures** alongside the JWT. The holder presents the JWT plus only the disclosures they choose; "with an SD-JWT document, a Holder can decide which claims to release" ([15], §1). "Prove I am John Smith without revealing more" therefore works by disclosing only the `given_name`/`family_name` disclosures and withholding all others; the verifier recomputes each disclosure's digest and matches it against digests embedded in the signed JWT — the issuer's signature still covers everything.

**Key Binding** is the second half: the SD-JWT payload may contain a `cnf` (confirmation) claim per RFC 7800; the presentation then includes a **Key Binding JWT (KB-JWT)** signed by the holder's `cnf` key over a verifier nonce, audience, timestamp and a hash of the presented SD-JWT+disclosures. "cnf: OPTIONAL unless cryptographic Key Binding is to be supported… For proof of cryptographic Key Binding, the KB-JWT in the presentation of the SD-JWT MUST be secured by" the cnf key ([15], SD-JWT VC draft §3.2.2.2 wording as fetched, lines defining `cnf`).

### 4.2 SD-JWT VC — still a draft

**SD-JWT-based Verifiable Credentials (draft-ietf-oauth-sd-jwt-vc)** profiles SD-JWT into a credential format (media type `dc+sd-jwt`), formalizes the Issuer-Holder-Verifier model ([15], §1.1), and defines Key Binding as "the Holder to prove they are the intended Holder" (§1.2). As of mid-2026 its IESG state is **"Publication Requested"** — approved by the WG but **not yet an RFC** ([15], datatracker header). This is the format the EUDI ecosystem and HAIP bet on; its non-final status is a (modest) standards-maturity risk.

### 4.3 mdoc / ISO/IEC 18013-5

ISO/IEC 18013-5 (mobile driving licence) defines the **mdoc** format: the issuer signs a **Mobile Security Object (MSO)** containing salted digests of each data element (structurally the same trick as SD-JWT, in CBOR/COSE), plus the **DeviceKey** — the holder-binding public key. At presentation the wallet returns only requested elements; the reader validates element digests against the issuer-signed MSO, and validates a **device signature** computed with DeviceKey over the **SessionTranscript**, binding the response to this session and device. Apple's verifier documentation confirms the mechanics: "The session transcript is used as the 'info' parameter during HPKE decryption, and is used again while verifying the device signature as defined in ISO 18013-5," and trust is rooted in **IACA (issuing authority certificate authority) certificates** ([12], FAQ). Namespaces (`org.iso.18013.5.1`, AAMVA, `org.iso.23220.1` for photo IDs) partition elements ([12]). Privacy features include age-over-N booleans ("if you only need to verify whether the user is above or below a certain age, you should use an age threshold element" [12]).

**Comparison:** SD-JWT and mdoc both give *issuer-controlled, hash-based selective disclosure with holder key binding*; neither gives unlinkability across presentations (the issuer signature is a stable correlator) — that is what BBS/ZKP work (ARF Topic G, TS4 "Specification of Zero-Knowledge Proof Implementation in EUDI Wallet" [5]) targets.

---

## 5. Wallet Landscape

### 5.1 EUDI Wallet (ARF v2.9.0, 21 May 2026)

The Architecture and Reference Framework is "the common technical baseline" for the wallets every Member State must offer under the amended eIDAS Regulation (EU) 910/2014 / 2024/1183 ([5], eudi.dev front page). Notable architecture facts (ARF main document [5a]):

- The ARF itself "is **informative** and intended to support implementation; it does not" bind legally — normative force comes from the Commission Implementing Regulations (CIR series listed in §1.5).
- Keys live in a **Wallet Secure Cryptographic Device (WSCD)** accessed via a Wallet Secure Cryptographic Application (WSCA); "a Wallet Instance can access a WSCD that has a level" of certified security, and "a **Key Attestation (KA)** attests that a WSCA/WSCD or keystore used by the Wallet Unit complies with the relevant requirements. A Key Attestation contains public keys whose corresponding private keys are managed by that WSCA/WSCD" ([5a], §3.4/§4.5 region).
- **PID (Person Identification Data) is issued and managed at Level of Assurance (LoA) High**; "the LoA concept only applies to PID," not to other attestations ([5a], §2/§3 narrative). PID Providers verify "the identity of the User in compliance with LoA high requirements" ([5a], §3.4).
- Formats/protocols (ARF §5.4 "Technical attestation formats and proof mechanisms," per ToC; corroborated by the PID Rulebook, now moved to the attestation-rulebooks catalog on GitHub [5b]): SD-JWT VC and ISO/IEC 18013-5 mdoc, presented via OpenID4VP and ISO 18013-5; remote presentation flows reference "transactional data using ISO/IEC 18013-5 and OpenID4VP" ([5a], §5.7.2). *(Portion of §5 beyond the fetch truncation; flagged.)*
- Relevant open topics: Topic F (Digital Credentials API), Topic Ab (**digital signature using wallet** — QES creation is an explicit EUDI wallet function), Topic Z (**device-bound attestations**), TS3 (Wallet Unit Attestation), TS12 (Strong Customer Authentication with the wallet) ([5]).

**Who can verify?** Any registered **Relying Party**: RPs register with Member State registrars and authenticate with certificates from Access CAs listed in a signed List of Trusted Entities; there is deliberately "no Trusted List… for Relying Parties" because "the expected number of Relying Parties" is too large ([5a], §3.5, §6.4.2).

### 5.2 Apple Wallet (mDL + Digital ID)

Apple Wallet holds ISO 18013-5 mdocs: US driver's licenses/state IDs (15+ states and Puerto Rico as listed), the Japan My Number national ID (ISO 23220 namespaces), and — since iOS 26.1 — **"Digital IDs using a U.S. passport,"** for which *Apple itself* is the IACA issuer ("IACA certificates for Apple") ([12]).

- **Assurance at issuance:** "the user must prove ownership of a valid government issued ID card. During this proofing process, the issuing authority confirms that the user's ID card is authentic and belongs to that user"; presentation "requires the same Face ID or Touch ID that they used to add the ID card" ([12]). This is government-grade proofing (roughly NIST IAL2-equivalent, delegated to the DMV/State Dept process), but Apple publishes no formal IAL/LoA mapping.
- **Who can verify:** in-app via the **Verify with Wallet API**, which is **entitlement-gated** — granted per bundle ID, restricted to enumerated categories (air travel, financial services, government, healthcare, car rental, etc.) ([12]). For the web: "For browser-based online presentment, we intend to support the mDoc request API as developed in the W3C… in a way that also enables presentment of conforming identity credentials from third party applications" ([12], FAQ) — i.e., DC-API-mediated web verification is coming but was not GA in this document.

### 5.3 Google Wallet

Google Wallet ID passes are "built on an open ISO standard" (ISO 18013-5); an ID "is only added when the issuer approves" after the user "submit[s] information to the issuer to prove their identity"; verification = "checking the cryptographic signature" against issuer certificates ([13]). Documentation covers in-person (NFC/QR) acceptance and **online acceptance** ("Verify with Google Wallet — Present credentials on Android-powered devices"; provisioning docs for third-party issuers) via the Android Credential Manager / DC API path ([13]). Same caveat: no published IAL/LoA mapping; assurance inherits from the issuing authority's proofing.

### 5.4 Microsoft Entra Verified ID

Entra Verified ID is an issuance/verification service for W3C VCs "signed with the `did:web` method," with Microsoft Authenticator as wallet, and the older DIF stack (SIOPv2, Presentation Exchange, Well-Known DID Configuration) for presentation ([9]). **Assurance at issuance is whatever the tenant's onboarding provides** — the documented flow issues a VC after an ordinary IdP sign-in: "The sign-in step uses traditional authentication (such as username and password) to verify the employee's identity with the issuer" ([9]). So Verified ID is an *enterprise attribute attestation* system (employment, membership), not an inherent government-grade identity system; higher assurance (e.g., Face Check biometric matching against the VC portrait) is an add-on service (not fetched/verified in this pass). **Who can verify:** anyone — verification uses open standards or Microsoft's Request Service API; trust in the issuer is via its did:web domain ([9]).

---

## 6. Trust Registries & Issuance Assurance

### 6.1 Registry models

- **EU Trusted Lists (eIDAS Art. 22) / ARF trust model.** "A Trusted List or LoTE Provider is a body responsible for maintaining, managing, and publishing Trusted Lists and/or Lists of Trusted Entities." Trusted Lists exist for Wallet Providers, PID Providers, and Attestation Providers; entries carry the entities' **trust anchors** (certificates); entities "must be notified to the Commission by a Member State" to be listed ([5a], §3.5). The European Commission publishes a List of Trusted Lists so "find all Trusted Lists and LoTEs in the ecosystem" is bootstrapped centrally ([5a], §6.2.2 region). This is *accreditation-based* trust: being on the list is the authorization.
- **mdoc/IACA model.** Verifiers maintain a set of IACA root certificates per issuing authority (Apple publishes per-state IACA download links; "In order to authenticate information returned from IDs in Wallet, your server will need to trust these IACA certificates" [12]). Trust distribution is ad hoc — each verifier curates roots; VICAL (ISO 18013-5 trusted-list format) exists but adoption is verifier-by-verifier.
- **did:web / domain-reputation model** (Entra): the issuer's DID is its domain; "a permission-based model that allows trust using a web domain's existing reputation" ([9]) plus DIF Well-Known DID Configuration for domain linkage.
- **OpenID Federation / ecosystem profiles**: HAIP leaves trust-framework selection to ecosystems and requires them to specify "which Key Attestation format" and trust anchors apply ([7], §9.3).

### 6.2 Identity assurance at issuance

- **NIST SP 800-63A (rev. 4)** "provides requirements for the identity proofing of individuals at each identity assurance level (IAL) for the purposes of enrolling them into an identity service," with outcomes of resolution, **attribute validation** ("confirm the accuracy of the core attributes"), and verification; "each successive IAL builds on the requirements of lower IALs" ([16], Abstract, §1). IAL1/2/3 escalate evidence strength, validation, and (at IAL3) supervised biometric-backed proofing. *(The individual IAL definition paragraphs fell in fetch-omitted long lines; the framework structure quoted above was read directly.)*
- **eIDAS LoA (low / substantial / high)** per CIR (EU) 2015/1502; in the EUDI ecosystem "the LoA concept only applies to PID," and PID issuance requires LoA High identity verification ([5a]).
- The critical design fact: **assurance is a property of the issuance event, carried by the credential, not by the authentication method used later.** A VC issued at LoA High remains a LoA-High attribute assertion regardless of how the holder later authenticates — *provided holder binding is sound*, which is precisely the crux below.

---

## 7. Holder Binding & Passkey Linkage — Crux Analysis

### 7.1 How holder binding actually works

Across the stack, cryptographic holder binding has two halves:

1. **At issuance:** the wallet proves possession (or provides an attestation) of a key, and the issuer embeds that public key in the credential — `cnf` (JWK) in SD-JWT VC ([15]), DeviceKey in the mdoc MSO ([12] mechanics), or subject `id`/verification method in a W3C VC. OpenID4VCI's `proofs` parameter carries this: a `jwt` proof (self-signed PoP over `c_nonce` and issuer audience), or an **`attestation` proof type** in which "a proof type may provide the cryptographic public key(s) either with corresponding proof(s) of possession… or with **key attestation(s)**" ([3], §8.1, Appendix F). The Appendix D **key attestation** is a JWT from the wallet/keystore provider attesting a batch of keys and their storage/user-verification properties; HAIP makes it mandatory: "Wallets MUST support key attestations," with X.509-chained attestation signers ([7], §4.5.1). Separately, **wallet attestation** (Appendix E) authenticates the wallet *software* as an OAuth client; HAIP requires it and — important privacy point — "Wallet Attestations… MUST NOT introduce a unique identifier specific to a single Wallet instance" ([7], §4.4.1).
2. **At presentation:** the holder signs a verifier-chosen challenge with the bound key — KB-JWT for SD-JWT VC (over nonce, audience, and the hash of the presented credential+disclosures [15]), device signature over SessionTranscript for mdoc ([12]). OpenID4VCI §8.1 states the purpose: the verifier can check "that the End-User presenting a Credential is the same End-User to whom that Credential was issued" ([3]).

The specs also recognize **claims-based binding** (match name/DoB against another credential) and **no binding** ([3] §§14.1–14.2; [4] §5.3) — meaning PassSign cannot assume every credential in the wild is cryptographically holder-bound.

### 7.2 Can a passkey public key be the holder-binding key of a VC?

**Structurally, yes — practically, only with a new profile.** Analysis:

**(a) Expressibility: yes.** `cnf` accepts any JWK (RFC 7800 semantics [15]); a passkey's ES256/Ed25519 public key (COSE_Key → JWK conversion is lossless) can be placed in `cnf` at issuance. Nothing in SD-JWT VC or OpenID4VCI forbids it.

**(b) Signature format: the real blocker.** A WebAuthn assertion does **not** sign the raw challenge. As the WebAuthn `sign` extension proposal puts it: "a WebAuthn assertion signature… signs not over the `challenge` parameter provided by the RP or client, but over the concatenation of authenticator data and a hash of a JSON object embedding that challenge" ([17]). A KB-JWT must be a JWS whose signature covers exactly the JWS signing input; an mdoc device signature must cover exactly the SessionTranscript structure. A passkey cannot produce either. Therefore **a passkey can serve as a `cnf` key only if the verifier accepts a WebAuthn-shaped proof-of-possession**: e.g., a profile where `challenge = SHA-256(KB-payload)` and the verifier validates `authenticatorData || SHA-256(clientDataJSON)` and parses `clientDataJSON` to recover the challenge. This is exactly what practitioners already do as a workaround — on the sign-extension thread, Orie Steele (OR13) reports: "I've implemented hacks around Digital Credential wallets, **binding credentials to passkeys**, by proxying information in and out of frames, and **overloading the 'challenge' to sign arbitrary data**… It's all gross, this API would lead to a much better experience" ([17]). So passkey-as-holder-key is *proven feasible in the field* but has no standardized proof format — an open lane for PassSign.

**(c) The forward path: WebAuthn `sign` extension (w3c/webauthn PR #2078).** The proposal — authored by Emil Lundberg (Yubico) — "allows for signing arbitrary data using a key associated with but different from a WebAuthn credential key pair," with the first motivating use case being "attested, hardware-bound signing keys for applications such as **digital identity wallets and similar verifiable credentials**" ([17]). The signing key is deliberately distinct from the passkey assertion key so arbitrary signing "cannot be used to bypass the domain binding restrictions for WebAuthn credentials" ([17]). John Bradley (ve7jtb) frames the strategic intent: "What we are trying to do is create a standardized API for a WSCD… This gives FIDO/Passkeys an opening to provide the crypto functionality used by EUDI wallets. **Attestation is an important part of the value proposition**" ([17]). Status signals from the thread: Yubico had only "rough internal proof-of-concept prototypes," design iteration moved to `yubicolabs/webauthn-sign-extension`, and no other WG commitments were recorded ([17]) — i.e., **not shippable yet, but the standards venue and design intent align exactly with PassSign's thesis**.

**(d) Key attestation gap for synced passkeys.** OpenID4VCI/HAIP high-assurance issuance expects key attestations describing storage security. Security keys and some device-bound platform credentials can attest; **consumer synced passkeys generally return no meaningful attestation, and their private keys are provider-synced across devices** — which conflicts with EUDI's WSCD model, with ARF Topic Z (device-bound attestations), and with any "sole control" claim at QES level. A passkey-bound VC is therefore realistic at *substantial*-type assurance; *high*/QES-grade binding needs device-bound keys with attestation (or the sign extension's attested hardware-bound keys).

**(e) RP-ID scoping.** A passkey is exercisable only within its Relying Party ID scope. A VC whose `cnf` key is a passkey can produce a PoP **only in a ceremony mediated by that RP** — incompatible with the open "present to any verifier" wallet model, but **perfectly compatible with an e-signature service model**, where one service (the PassSign platform / the document platform) is the fixed RP that orchestrates every signing ceremony and where verifiers validate *evidence records* rather than run live ceremonies. This asymmetry is the single most important architectural observation of this stream.

### 7.3 Bottom line on the crux

- Passkey public key **in** a VC `cnf`: expressible today; no interop profile exists.
- Passkey **signing** a standard KB-JWT/mdoc device response: impossible today (signature format); possible via challenge-overloading profile (deployed as hacks, standardizable), or cleanly via the WebAuthn `sign` extension (pre-standard, prototypes only, explicitly motivated by VC holder binding and EUDI WSCD [17]).
- The inverse composition is available **now**: use OpenID4VP/DC API to obtain a verified-identity presentation (SD-JWT VC KB-JWT or mdoc device signature with the *wallet's* key), and bind that event to a passkey (WebAuthn assertion over a challenge derived from the same session/document hash) in one evidence record. Identity assurance comes from the credential; signing continuity comes from the passkey. No new cryptography required — only an evidence-record specification.

---

## 8. Limitations Register

| # | Limitation | Evidence | Severity for PassSign |
|---|---|---|---|
| L1 | WebAuthn cannot sign arbitrary data; KB-JWT/mdoc PoP formats are incompatible with assertion signatures | [17] PR #2078 problem statement | **High** — blocks direct passkey-as-holder-key without a new profile |
| L2 | WebAuthn `sign` extension is pre-standard: prototypes only, no browser commitments recorded | [17] emlun comments | High — cannot be a near-term dependency |
| L3 | Synced passkeys lack key attestation and are multi-device by design; conflicts with WSCD/device-bound-key requirements (ARF Topic Z) and QES sole-control | [3] App. D, [7] §4.5.1, [5] | High for eIDAS-high/QES ambitions; moderate otherwise |
| L4 | Passkey RP-ID scoping restricts PoP ceremonies to one relying party | WebAuthn design, discussed in [17] | Moderate — fine for a centralized signing service, fatal for open-wallet PoP |
| L5 | SD-JWT VC is still an Internet-Draft (IESG: Publication Requested) | [15] datatracker | Moderate — format churn risk |
| L6 | Digital Credentials API is a W3C Working Draft (16 June 2026); protocol registry still moving; Apple web presentment "intended," not shipped in fetched docs | [6], [12] | Moderate — web ceremony availability varies by browser/region |
| L7 | Wallet ID verification is gated: Apple entitlements are per-app and category-restricted; EUDI relying parties must register with Member State registrars | [12], [5a] §3.5/§6.4.2 | High — a PassSign verifier cannot assume free access to wallet IDs |
| L8 | No public IAL/LoA certification for Apple/Google Wallet IDs; Entra Verified ID assurance = tenant's own proofing (may be just username/password) | [12], [13], [9] | Moderate — assurance labeling must be per-issuer |
| L9 | Holder binding is optional in the specs (claims-based/no-binding presentations exist) | [3] §§14.1–14.2, [4] §5.3 | Moderate — PassSign policy must require cryptographic binding explicitly |
| L10 | Status/revocation checking creates issuer phone-home and only herd privacy (131k-entry lists) | [8] §§2–3 | Low–moderate (privacy/availability of long-lived signature evidence) |
| L11 | DIDs offer no adoption leverage: EUDI/mdoc ecosystems are X.509-based; Entra uses did:web only | [5a], [7], [9], [12] | Low — avoid DID dependency |
| L12 | Research-method limits: WebSearch unavailable; large specs truncated at ~100 KB (OID4VCI appendices, ARF §5+, NIST IAL paragraphs verified indirectly); FIDO Alliance digital-credentials page returned empty | this document | — (documented) |

---

## 9. Opportunities for PassSign

1. **Define the missing profile: "WebAuthn proof-of-possession for VC holder binding."** A short spec that (i) puts a passkey JWK in `cnf` at issuance (OID4VCI `jwt`-proof variant where challenge = hash of the PoP payload), and (ii) defines a KB-JWT-equivalent whose signature is a WebAuthn assertion (verifier recomputes challenge from the presentation hash, validates clientDataJSON/authenticatorData). Practitioners already do this as "gross" hacks ([17]) — standardizing it is a genuinely open, low-crypto-risk contribution, and the OpenID DCP WG's proof-type extension point ("the proof type… is an extension point that enables the use of different types of proofs" [3] §8.2) is the designed venue.
2. **Exploit the RP-scoping asymmetry.** PassSign's e-signature model has a *fixed ceremony RP* — the exact setting where passkey-bound credentials work despite L4. Position PassSign evidence records (document hash + VP + WebAuthn assertion over the same hash) as the verifiable artifact, so third parties verify records, not live ceremonies.
3. **Compose, don't wait:** identity-at-signing via OpenID4VP/DC API presentations (SD-JWT VC / mdoc) + signature-continuity via passkey, both bound to the document hash in one evidence structure. Every component on this path is final/Recommendation status today (OID4VP Final [4], VCDM 2.0 / Bitstring REC [1][8], SD-JWT RFC 9901 [14]) except the browser API (WD [6]).
4. **Track and engage the WebAuthn `sign` extension** ([17]) — it is explicitly aimed at making passkey-managed, attested, hardware-bound keys usable for credentials and EUDI WSCD purposes; PassSign is a named-shaped use case and could supply implementation experience/requirements (e.g., signing document hashes for AdES-style signatures).
5. **Assurance labeling layer.** Since wallets don't expose uniform IAL/LoA (L8), PassSign should record *issuer + credential type + trust anchor* in evidence and map to assurance levels via configurable trust lists — mirroring the ARF Trusted List pattern ([5a] §3.5) rather than inventing a registry.
6. **EUDI alignment for the EU market:** support SD-JWT VC + mdoc verification via OpenID4VP, X.509 trust anchors from EU Trusted Lists, and monitor ARF Topic Ab ("digital signature using wallet") — the EUDI wallet's built-in QES function is both a competitor and an interop target for PassSign.

---

## 10. Sources

1. W3C, *Verifiable Credentials Data Model v2.0* (W3C Recommendation) — §Conformance, §Terminology, §4.12 Securing Mechanisms, information-graph figures. https://www.w3.org/TR/vc-data-model-2.0/
2. W3C, *Decentralized Identifiers (DIDs) v1.0* (W3C Recommendation, 19 July 2022) — Abstract, §1, §5 (verification methods). https://www.w3.org/TR/did-1.0/
3. OpenID Foundation, *OpenID for Verifiable Credential Issuance 1.0* (Final) — §2 (Holder Binding definitions), §3.1, §7.2 (`c_nonce`), §8.1–8.2, §§14.1–14.2, §15.4.4, Appendices D (Key Attestations), E (Wallet Attestations), F (Proof Types: `jwt`, `di_vp`, `attestation`). https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html *(fetched text truncated before appendices; appendix contents confirmed via spec ToC and [7].)*
4. OpenID Foundation, *OpenID for Verifiable Presentations 1.0* (Final) — §1, §2, §3.1–3.2, §4, §5.1, §5.3, §6 (DCQL), Appendix A (DC API). https://openid.net/specs/openid-4-verifiable-presentations-1_0.html
5. EU Digital Identity Wallet, *ARF portal* (v2.9.0, 21 May 2026) — landing page, Discussion Topics (F, Ab, Z, G), Technical Specifications TS1–TS12. https://eudi.dev/latest/ (canonical: https://eu-digital-identity-wallet.github.io/eudi-doc-architecture-and-reference-framework/2.9.0/)
   - 5a. *ARF Main Document* — §1.3 (informative status), §3.4 (Wallet Provider, Key Attestation, WSCA/WSCD), §3.5 (Trusted List or LoTE Provider), §5.7.2, §6.2–6.4 (notification, registration, Access CA LoTE), LoA-High PID statements. https://eudi.dev/latest/architecture-and-reference-framework-main/ *(read to ~§6 due to truncation).*
   - 5b. *ARF Annex 3.01 PID Rulebook* (moved to attestation-rulebooks catalog). https://eudi.dev/latest/annexes/annex-3/annex-3.01-pid-rulebook/ and https://github.com/eu-digital-identity-wallet/eudi-doc-attestation-rulebooks-catalog/blob/main/rulebooks/pid/pid-rulebook.md
6. W3C Federated Identity WG, *Digital Credentials API* — W3C Working Draft, 16 June 2026 — Status, §7 (DigitalCredential interface, protocol enumerations, `navigator.credentials.get()/create()`). https://www.w3.org/TR/digital-credentials/
7. OpenID Foundation, *OpenID4VC High Assurance Interoperability Profile 1.0 (HAIP)* — §3, §4.4.1 (Wallet Attestation, x5c, no per-instance identifiers), §4.5.1 (Key Attestation mandatory; Appendix D format; `jwt` proof with `key_attestation`), §7 (status), §9.2–9.3. https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-1_0.html
8. W3C, *Bitstring Status List v1.0* — W3C Recommendation, 15 May 2025 — §2 (statusPurpose, statusListIndex, encodedList), §3 (131,072 minimum entries, herd privacy), §4 (algorithms). https://www.w3.org/TR/vc-bitstring-status-list/
9. Microsoft, *Introduction to Microsoft Entra Verified ID* (Learn, updated 2026-03-16) — did:web trust system, Authenticator wallet, SIOP/Presentation Exchange, issuance-via-ID-token note. https://learn.microsoft.com/en-us/entra/verified-id/decentralized-identifier-overview
10. W3C CCG, *did:key Method* (community specification). https://w3c-ccg.github.io/did-key-spec/ *(not fetched; definitional reference only.)*
11. *did:jwk Method Specification*. https://github.com/quartzjer/did-jwk/blob/main/spec.md *(not fetched; definitional reference only.)*
12. Apple Developer, *Get started with the Verify with Wallet API* — availability (incl. Digital ID via U.S. passport, iOS 26.1), entitlement categories, ISO/IEC 18013-5 & AAMVA & ISO 23220 namespaces, IACA certificates (incl. Apple as IACA for Digital ID), session transcript/HPKE/device-signature verification, W3C mDoc-request-API intent. https://developer.apple.com/wallet/get-started-with-verify-with-wallet/
13. Google, *Google Wallet — Identity* — ISO-standard IDs, issuer-approved provisioning, signature-based verification, in-person and online acceptance. https://developers.google.com/wallet/identity
14. IETF, *RFC 9901: Selective Disclosure for JWTs (SD-JWT)* — Proposed Standard, November 2025. https://datatracker.ietf.org/doc/rfc9901/ (text: https://www.rfc-editor.org/rfc/rfc9901.html)
15. IETF, *SD-JWT-based Verifiable Credentials* (draft-ietf-oauth-sd-jwt-vc; IESG state: Publication Requested as of fetch date) — §1.1 Issuer-Holder-Verifier Model, Key Binding/KB-JWT, `cnf` (RFC 7800). https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/
16. NIST, *SP 800-63A (rev. 4): Digital Identity Guidelines — Identity Proofing and Enrollment* — Abstract, Expected Outcomes, Identity Assurance Levels. https://pages.nist.gov/800-63-4/sp800-63a.html *(IAL definition paragraphs fell in fetch-omitted lines; framework text quoted was read directly.)*
17. W3C WebAuthn WG, *PR #2078: Add "sign" extension* (Emil Lundberg/Yubico; discussion incl. O. Steele on passkey-credential-binding hacks, J. Bradley on EUDI WSCD/attestation; iteration repo yubicolabs/webauthn-sign-extension) — May 2025 preview. https://github.com/w3c/webauthn/pull/2078
18. FIDO Alliance, *Digital Credentials* page — **fetch returned empty content**; no claims sourced. https://fidoalliance.org/digital-credentials/
19. European Commission, Regulation (EU) 910/2014 as amended (consolidated) — eIDAS/EUDI legal basis. https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:02014R0910-20241018 *(cited via ARF portal; not fetched.)*
