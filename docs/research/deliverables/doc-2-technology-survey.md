# PassSign Document 2 — Technology Survey

**Phase 1 deliverable (Streams 1–3: WebAuthn/Passkeys · Digital Identity · Digital Signature Standards)**
Version 0.1 · 2026-07-17 · PassSign Research Program

This document answers primary questions **Q1 (Is it technically possible?)** and, preliminarily, **Q3 (Is it interoperable?)**. It consists of this synthesis followed by the three full stream research documents as Parts I–III. All normative claims in the parts are cited to primary sources; claims that could not be verified against a fetched primary source are marked **[unverified]**.

---

## Executive synthesis

### The one-sentence finding

**Passkeys cannot sign documents — but they can authorize signatures, and every component needed to compose that authorization into a legally recognizable, standards-conformant signature already exists.**

### The pivotal technical constraint

A WebAuthn assertion signs exactly `authenticatorData || SHA-256(clientDataJSON)` (WebAuthn L3 §6.3.3; Part I §2). RP-chosen bytes enter only via the `challenge` field, doubly wrapped (base64url → JSON → SHA-256), and the signed `type` is always `"webauthn.get"`. The cryptographic statement a passkey makes is therefore always *"I authenticated to this origin"* — never *"I signed this document."* No standard signature container (CMS `SignedAttributes` per RFC 5652 §5.4, JWS signing input per RFC 7515 §5.1, mdoc SessionTranscript) can be produced by a WebAuthn authenticator as deployed (Part III §4.1).

### Three paths assessed (Gate 1 decision input)

| Path | Mechanism | Feasibility today | Evidence quality | Verdict |
|---|---|---|---|---|
| **(a) Challenge stuffing** | `challenge` = document hash/manifest; archive assertion as evidence | Works in every browser now; vendor-documented; deployed as acknowledged hacks | Weak: no WYSIWYS, authentication semantics, key entanglement, nonstandard envelope, no attestation on synced passkeys | Acceptable only as SES-grade evidence hardening; cannot be hardened past its structural limits (Part I §9a) |
| **(b) WebAuthn `sign` extension** | w3c/webauthn PR #2078: derived, attestable signing keys distinct from the credential key; signs raw input | Pre-standard: Yubico prototypes only, no browser commitments, Level 4 FPWD milestone, L4 ≈ Q4 2028 | Architecturally correct: fixes payload, semantics, key-separation, and (with attested keys) attestation | **Track and shape; do not build on before ~2029–2030** (Part I §9b, Part III §4) |
| **(c) Passkey-authorized signing key** | Passkey assertion (UV=1, challenge binds document manifest) activates a separately held signing key (HSM/RSSP); output is standard CAdES/PAdES/JAdES | Buildable now entirely from existing standards (CSC API v2.2, ETSI TS 119 432 V1.3.1, CEN EN 419 241-2) | Strong: byte-perfect standard signatures that validate in Adobe/EU DSS today; mirrors the remote-QSCD pattern regulators already accept | **Recommended baseline** (Part I §9c, Part III §10 Track A) |

Streams 1 and 3 converge independently on the same recommendation: **path (c1) — server/HSM-held signing key activated by a fresh passkey assertion — is the PassSign baseline**, with (a) as a hardened evidence profile and (b) as the future native path the protocol abstraction must accommodate.

### The identity layer (Stream 2) composes cleanly — with one asymmetry to exploit

Verified identity comes from the credential stack, not the passkey: OpenID4VP presentations of SD-JWT VC (RFC 9901 base) or ISO 18013-5 mdocs, delivered via the browser Digital Credentials API (W3C WD, shipping in Chromium). A passkey public key *can* be expressed as a VC holder-binding key (`cnf` accepts any JWK) but cannot produce a standard KB-JWT/mdoc proof — the same payload constraint again (Part II §7.2). Two consequences:

1. **The composition available now:** obtain an identity presentation (wallet-signed) and a passkey assertion over the same document-bound challenge, and bind both into one evidence record. Identity assurance comes from the credential; signing continuity comes from the passkey. No new cryptography — only an evidence-record specification (Part II §7.3).
2. **The RP-scoping asymmetry:** passkeys only work within their Relying Party ID scope — fatal for the open "present anywhere" wallet model, but *perfectly compatible* with a signing-service model where one fixed RP orchestrates ceremonies and third parties verify evidence records rather than run live ceremonies (Part II §7.2e). This is the single most important architectural observation of Phase 1.

Also confirmed: DIDs offer no adoption leverage (EUDI ARF and mdoc ecosystems are X.509/Trusted-List based; Entra retreated to did:web); PassSign should be X.509-first with DID methods as optional wrappers (Part II §2.2).

### Cross-stream limitations that shape the protocol

1. **Synced passkeys provide no attestation and replicate keys via provider escrow** (Part I §5) — they cannot anchor sole-control or key-provenance claims. Authenticator class (synced vs device-bound) must be a recorded, first-class protocol attribute.
2. **No algorithm identifier exists for WebAuthn-shaped signatures** in JOSE/COSE/CMS registries; unknown algorithms yield INDETERMINATE/FAILED in all deployed validators (Part III §7). Direct-assertion evidence therefore requires a new profile (JAdES is the closest container; Part III §3).
3. **No X.509 certificate can currently be issued for a passkey key**: plain credentials cannot sign a PKCS#10 CSR (proof-of-possession); attestation-based issuance (ACME device-attest-01 pattern) is the viable route but consumer passkeys often attest `none` (Part III §5).
4. **WYSIWYS does not exist** anywhere in the WebAuthn stack; the only browser-rendered precedent is Secure Payment Confirmation, payments-specific and Chromium-bound (Part I §4.3). The signing ceremony's display guarantees must come from the protocol/RP layer.

### Prior-art vacuum (decision-relevant negative findings)

No GitHub project, academic paper, IETF draft, or standards work item was found that packages WebAuthn assertions into CMS/PDF/JAdES containers or specifies a passkey-native document-signing protocol (Part III §4.3; confirmed independently by Stream 6's collision scan). Practitioners' challenge-overloading hacks exist and are described as "gross" by their own authors in the WebAuthn WG (Part II §7.2b). The composition space PassSign targets is genuinely unoccupied.

### Scorecard

| Question | Verdict | Basis |
|---|---|---|
| **Q1 — Technically possible?** | **Supported, with a precise qualification**: not by direct passkey signing (structurally excluded as deployed), but by a defined ceremony composing passkey authorization with standard signature creation — path (c) now, path (b) natively later | Parts I–III |
| **Q3 — Interoperable? (preliminary)** | **Supported on the baseline path** (output is byte-perfect PAdES/CAdES/JAdES validating in existing tooling); **profile work required** for direct-assertion evidence (new JOSE alg / CMS OID, JAdES extension) | Part III §§4, 7 |

### Gate 1 recommendation

Adopt a **primitive-agnostic ceremony abstraction** with three conformance classes — (a) assertion-evidence, (b) native sign-extension, (c) authorized-signing-key — and make **(c1) the baseline profile**. This keeps the protocol shippable now, upgradeable when PR #2078 lands, and honest about the evidence quality of each class.

---

*The three parts follow. Each retains its own limitations register, opportunities analysis, and numbered sources.*

---

# Stream 1: WebAuthn & Passkeys

**PassSign research program — Stream 1 of 6. Date: 2026-07-17.**

Scope: Can the WebAuthn/passkey stack, as specified and as actually deployed, produce signatures over documents — and what are the hard constraints? Normative claims cite the W3C WebAuthn Level 3 spec (Candidate Recommendation track, single-page TR at w3.org) [1], WebAuthn Level 2 (Recommendation) [2], and primary WG artifacts. Section numbers below were verified against the live Level 3 TR table of contents on 2026-07-17. Where a normative source could not be fetched in full (the TR is a single ~1.5 MB page and fetches truncated), the claim is standard spec text known to the author and the section anchor is verified; genuinely uncertain items are marked **[unverified]**.

---

## 1. Architecture

WebAuthn is a W3C-standardized challenge–response public-key authentication API with four actors:

- **Relying Party (RP):** the web service. Identified by an **RP ID**, a domain string (registrable-domain suffix of the origin). Credentials are *scoped* to the RP ID; an authenticator will refuse to use a credential for any other RP ID ([1] §4 Terminology, "Credential Key Pair"; "Relying Party Identifier").
- **Client (browser/OS platform):** mediates everything. The RP never talks to the authenticator; it calls `navigator.credentials.create()` (registration ceremony, [1] §7.1) or `navigator.credentials.get()` (authentication ceremony, [1] §7.2), and the client validates the origin, computes the RP ID, constructs the `clientDataJSON`, renders all UI, and invokes the authenticator operations `authenticatorMakeCredential` ([1] §6.3.2) and `authenticatorGetAssertion` ([1] §6.3.3).
- **Authenticator:** platform (built into the OS/device, e.g., iCloud Keychain, Google Password Manager, Windows Hello) or roaming (e.g., a YubiKey over USB/NFC/BLE, speaking FIDO CTAP2 [10]). Generates and holds the credential private key; the private key "is expected to never be exposed to any other party, not even to the owner of the authenticator" ([1] §4, "Credential Private Key").
- **User:** supplies a **test of user presence (UP)** — a physical gesture such as a touch — and optionally **user verification (UV)** — PIN, passcode, or biometric performed locally by the authenticator. UV is the distinction between "someone touched the key" and "the enrolled user unlocked it". Both facts are reported to the RP as single bits in the signed authenticator data (below); the RP requests UV via `userVerification: "required" | "preferred" | "discouraged"` ([1] §5.8.6).

**Registration** creates a fresh key pair per (RP, account), returns the public key plus (optionally) an **attestation statement** signed by a key that vouches for the authenticator model ([1] §6.5). **Authentication** returns an **assertion signature** made with the credential private key. A **passkey** is, per the L3 terminology, a *client-side discoverable credential* — one usable without the RP supplying the credential ID, typically synced across devices by a credential provider ([1] §4, "Passkey"; §6.1.3 Credential Backup State).

Key architectural fact for PassSign: **the RP supplies exactly one attacker-controlled... rather, RP-controlled input into the signed payload — the `challenge` byte string.** Everything else in the signed bytes is produced by the client and authenticator.

## 2. The Signed Payload (byte level)

This is the pivotal constraint. In an authentication ceremony, the authenticator does **not** sign the challenge. Per [1] §6.3.3 (The authenticatorGetAssertion Operation), the assertion signature is computed over:

```
toBeSigned = authenticatorData || SHA-256(clientDataJSON)
```

i.e., the concatenation of the raw authenticator-data byte structure and the client data hash. (The registration/attestation signature in `packed` format is likewise over `authenticatorData || clientDataHash`, [1] §8.2.) Signature encoding for ES256 is ASN.1 DER ECDSA-Sig-Value, per [1] §6.5.5 (Signature Formats for Packed Attestation, FIDO U2F Attestation, and Assertion Signatures).

### 2.1 authenticatorData ([1] §6.1, anchor `#sctn-authenticator-data`)

| Offset | Length | Field | Content |
|---|---|---|---|
| 0 | 32 | `rpIdHash` | SHA-256 of the RP ID (e.g., `SHA-256("example.com")`) |
| 32 | 1 | `flags` | Bit field, see below |
| 33 | 4 | `signCount` | 32-bit unsigned big-endian signature counter ([1] §6.1.1) |
| 37 | var | `attestedCredentialData` | Only present at registration (AT flag): 16-byte AAGUID ‖ 2-byte credentialIdLength ‖ credentialId ‖ credentialPublicKey (COSE_Key, CTAP2 canonical CBOR) ([1] §6.5.1) |
| var | var | `extensions` | CBOR map of authenticator extension outputs (ED flag) |

Flags byte: bit 0 = **UP** (user present); bit 2 = **UV** (user verified); bit 3 = **BE** (backup eligible — credential is a multi-device/synced credential); bit 4 = **BS** (backup state — currently backed up); bit 6 = **AT** (attested credential data included); bit 7 = **ED** (extension data included). BE/BS were added in Level 3 ([1] §6.1.3).

Note the consequences: the signed bytes vary per ceremony (`signCount`, flags, extension outputs), the signature always covers the RP ID hash (domain binding), and it always covers the UP/UV bits — the signature itself is evidence of whether user verification occurred.

### 2.2 clientDataJSON ([1] §5.8.1, `CollectedClientData`, anchor `#dictionary-client-data`)

Constructed **by the client, not the RP**, as a JSON serialization with members:

- `type`: `"webauthn.create"` (registration) or `"webauthn.get"` (authentication). Fixed by the client; an RP cannot make an assertion say anything else.
- `challenge`: base64url encoding of the `challenge` buffer the RP passed in.
- `origin`: the web origin the client observed (e.g., `https://sign.example.com`).
- `topOrigin` (L3): top-level origin when invoked cross-origin in an iframe; `crossOrigin`: boolean.

The RP receives the exact `clientDataJSON` bytes and must hash them itself for verification ([1] §7.2 steps). Because the challenge is embedded base64url-encoded inside JSON that is then hashed, **the document bytes (or hash) never appear directly in the signed message — only inside `SHA-256(JSON{...challenge...})`, concatenated after 37+ bytes of authenticator data.** Any verifier of such a signature must possess and archive the full `clientDataJSON` and `authenticatorData` artifacts, not just the document ([6] confirms this operationally).

Security consideration [1] §13.4.3 (Cryptographic Challenges) requires challenges to be randomly generated, ≥16 bytes, to prevent replay — the spec's own framing of `challenge` is as an anti-replay nonce, not a message-to-be-signed.

## 3. Algorithms

The RP requests algorithms at registration via `pubKeyCredParams` using **COSE Algorithm Identifiers** ([1] §5.8.5, `COSEAlgorithmIdentifier`; IANA COSE Algorithms registry [11]):

| COSE alg | Name | Notes on real support |
|---|---|---|
| -7 | ES256 (ECDSA P-256/SHA-256) | Universal. The only algorithm guaranteed everywhere; Apple platform authenticator is ES256-only **[unverified for latest OS releases]** |
| -257 | RS256 (RSASSA-PKCS1-v1_5/SHA-256) | Windows Hello TPM-backed credentials; registered for WebAuthn via RFC 8812 [11] |
| -8 | EdDSA (Ed25519) | YubiKey 5 (firmware 5.2+) and some software authenticators; not offered by Apple/Google platform authenticators **[unverified]** |
| -35/-36 | ES384/ES512 | Spec has test vectors ([1] §16.8–16.9) but authenticator support is rare |

L3 test vectors also cover Ed448 ([1] §16.12). No post-quantum algorithms are deployed; FIDO/CTAP PQC (ML-DSA) work is in progress but pre-standard **[unverified]**. Practical rule for PassSign: **ES256 is the only safe universal assumption**, and the RP has no way to force a specific algorithm at authentication time — the algorithm is fixed per credential at registration.

## 4. Extensions & Arbitrary-Data Signing

### 4.1 Can a document hash be injected via `challenge`? (challenge stuffing)

Mechanically, yes. Yubico's own developer documentation, "Using WebAuthn for Signing" [6], describes exactly this: replace the random challenge in `GetAssertion` with the hash of the file; archive `clientDataJSON` + `authenticatorData` + signature; verify with the registered public key. It works today with zero client or authenticator changes and any WebAuthn-supported algorithm.

But the semantics and the WG's own assessment cut against it:

1. **The signature does not say "I signed this document."** It says `type:"webauthn.get"` — "I authenticated to origin X" — with the document hash smuggled into a replay nonce. Emil Lundberg's sign-extension PR states the distinction explicitly: a WebAuthn assertion "signs not over the `challenge` parameter provided by the RP or client, but over the concatenation of authenticator data and a hash of a JSON object embedding that challenge" [3].
2. **No WYSIWYS (what-you-see-is-what-you-sign).** The user sees a generic passkey/login prompt. The challenge is invisible; the client UI cannot display what is being authorized. Symmetrically, *every ordinary login is potentially a blind signature*: a malicious or compromised RP can set `challenge = SHA-256(contract)` during what the user believes is a login. This is inherent to the API — the ceremony UX is identical.
3. **Documented practitioner objection.** In the PR #2078 review thread, Orie Steele describes having implemented passkey-credential-binding for digital-credential wallets "by proxying information in and out of frames, and overloading the 'challenge' to sign arbitrary data... It's all gross" [3].
4. **Spec-level tension.** [1] §13.4.3 defines the challenge as a random anti-replay value; reusing it as a message channel forfeits WebAuthn's replay-protection reasoning and produces a signature envelope (`authData‖SHA-256(JSON)`) that no existing signature-container standard (CMS/CAdES/JAdES) recognizes as a ToBeSigned structure — a bespoke verification profile is unavoidable (relevant to Stream 3).
5. **Key-use entanglement.** The same key pair authenticates logins and "signs" documents, weakening non-repudiation arguments (a login transcript and a signature are cryptographically the same kind of object, distinguishable only by RP-side bookkeeping).

### 4.2 The dropped transaction-confirmation extensions (txAuthSimple/txAuthGeneric)

WebAuthn Level 1 defined `txAuthSimple` (authenticator displays a prompt string, and the display content is bound into the extension output) and `txAuthGeneric`. They were **removed in Level 2** by WG decision to drop extensions that "have not been implemented and are not testable" (w3c/webauthn issue #1386) [7]; Mozilla never implemented it (Bugzilla 1540778) [8]. The practical failure mode: the extension required a trusted display on the authenticator, which security keys don't have. A "Revised txAuthSimple" proposal (issue #2022) exists but has not been adopted [9]. CTAP2.0 carried `txAuthSimple` support; it was dropped from later CTAP revisions **[CTAP-side status not verified against the CTAP2.1/2.2 PDFs in this session]**. Lesson for PassSign: authenticator-rendered transaction display has already failed once in this ecosystem for lack of hardware and implementer interest.

### 4.3 Secure Payment Confirmation (SPC) — the browser-rendered precedent

W3C SPC [12] layers a *transaction confirmation ceremony* on WebAuthn: the **browser** (not the authenticator) renders a mandatory transaction dialog (amount, payee, instrument), and the transaction details are embedded into the client data of the resulting WebAuthn assertion (SPC uses a distinct client data type, `"payment.get"`, with a payment member carrying payee origin, amount/currency and instrument details **[exact member names not re-verified this session]**). Trust therefore rests on browser UI integrity rather than authenticator displays, and SPC additionally relaxes origin binding (a merchant origin may invoke a bank-scoped credential). SPC is shipped in Chromium **[current cross-browser status unverified; WebKit/Gecko support has historically been absent]**. SPC is the strongest existing precedent that a *ceremony with displayed semantics* can be standardized on top of an unmodified assertion format — the displayed data goes into `clientDataJSON`, exactly where a PassSign document-manifest could go.

### 4.4 Storage/derivation extensions

- **`largeBlob`** ([1] §10.1.5): stores opaque data (spec's example use: certificates) with a credential, readable at assertion time. Could carry a signing certificate binding the passkey public key to an identity — but support is patchy (CTAP2.1 `authenticatorLargeBlobs` security keys; iCloud Keychain passkeys support largeBlob since iOS 17 **[unverified]**; Google Password Manager support partial **[unverified]**).
- **`prf`** ([1] §10.1.4, backed by CTAP2 `hmac-secret`): derives per-credential, per-input 32-byte secrets after user verification. It enables *deterministic key derivation* in JS — a candidate for path (c) below — but the WG's own conclusion in the precursor discussions to the sign extension (PRs #1895/#1945, cited in [3]) was that keys derived this way "are never truly unextractable" and are "not hardware-bound as they are exposed to the client process".
- **`credProtect`** (CTAP2.1, registered WebAuthn authenticator extension): raises the bar for using a credential without UV/credential ID; relevant to preventing silent use of a signing credential.
- **Attestation types** ([1] §6.5.3): Basic, Self, AttCA, Anonymization CA, None; conveyance `none | indirect | direct | enterprise`. **Enterprise attestation** (L2+) can return individually-identifying attestation (e.g., serial number) but only for pre-configured browsers/authenticators — a managed-fleet feature, unusable for consumer signing. L3 also adds the ability to request attestation in *authentication* ceremonies (`attestation`/`attestationFormats` in `PublicKeyCredentialRequestOptions`) but authenticator support is scarce **[unverified]**.

### 4.5 The `sign` extension — deep dive (w3c/webauthn PR #2078)

**What it is.** An open PR by Emil Lundberg (Yubico), opened 2024-05-28, adding a `sign` client/authenticator extension [3]. Milestoned **L4-FPWD**, labeled @Risk/FeatureProposal (L4 FPWD expected ~Q4 2028) *(milestone/labels and charter timing per parallel Stream 6; PR content verified directly)*. Design iteration happens in **YubicoLabs/webauthn-sign-extension** [4] (branch `sign-extension-external`; editor's copy at yubicolabs.github.io/webauthn-sign-extension/#sctn-sign-extension), with changes cherry-picked back to the PR "when we're satisfied with the design" (emlun, in [3]). PR preview last updated 2025-05-19.

**Design (from the PR body [3] and repo history):**
- Signs **the given input unaltered** — no authenticatorData concatenation, no clientDataJSON wrapper. This is the explicit contrast with assertions drawn in the PR body.
- Uses a **signing key pair distinct from the WebAuthn credential key pair**, so "this arbitrary input cannot be used to bypass the domain binding restrictions for WebAuthn credentials" — i.e., key separation is the mechanism that makes arbitrary-input signing safe; a sign-extension signature can never be confused with a login assertion. *(The known-context note about "context/prefix separation" corresponds to this key-separation design plus CTAP-layer domain separation of the derived keys; the exact CTAP prefix mechanics were not verifiable this session — **[unverified detail]**.)*
- Registration-time input generates a signing key (algorithm negotiation via COSE identifiers); assertion-time input requests a signature. Per the repo's revision history: v2 (2025-04-07) renamed the to-be-signed input `phData` → `tbs`, removed `keyHandle` from the client extension output, and switched key-reference structures to `COSE_Key_Ref` from the COSE "two-party signing" draft family; v3 (2025-05-19) fixed CBOR map keys in authenticator-data references *(from repo release notes via search — **[not independently re-fetched]**)*.
- Supports **attested, hardware-bound signing keys** — the motivating use cases named in the PR are digital-identity wallets/verifiable credentials and "general-purpose digital signatures" with FIDO security keys, "with seamless interoperability with existing cryptographic protocols". A sibling `kem` extension for encryption is planned.
- Related IETF work: **draft-bradleylundberg-cfrg-arkg** (Asynchronous Remote Key Generation), an **active individual CFRG draft, revision -10, last updated 2026-02-27, expires 2026-08-31**, by Lundberg & Bradley [5]. ARKG lets a party derive unlinkable public keys for which the authenticator can later derive the private keys — the machinery for minting many signing keys from one passkey without round-trips.

**Reception in the thread [3]:** Mike Jones (selfissued): "I support this idea. Where are we on implementations?" — emlun (reply): only "rough internal proof-of-concept prototypes" at Yubico, "no commitments from other WG participants at this point". John Bradley (ve7jtb): frames it as "a standardized API for a WSCD" for the **EU Digital Identity Wallet** (ARF §4.2), noting the EU's key-storage certification requirements are met by "only a handful of mobile phones", that "attestation is an important part of the value proposition", and that the larger use is **native apps**, not the web. Orie Steele: strong support, describes current challenge-overloading hacks as "gross", and walks through JOSE/COSE hash-then-sign layering (ES256 pre-hash ambiguity, ML-DSA context strings). Nina Satragno (Google) points to WebKit's `remote-cryptokeys` explainer as possibly overlapping — a signal that Apple is exploring a *different* API shape for the same problem. Community comments raise EdDSA-curve/ZK-friendly algorithm needs for EUDI.

**Status assessment:** technically the right construction for PassSign (raw-input signing, key separation, attestation), but: zero browser implementations, zero shipped authenticator support, no multi-vendor commitment recorded in the PR, competing Apple explainer, and a Level 4 timeline (~FPWD Q4 2028, Rec years later). The "Signing additional data" scope item in the draft 2026–2028 WG charter *(per parallel Stream 6; the charter-draft URL timed out on fetch this session — **[not independently verified]**)* confirms intent but not delivery.

## 5. Synced vs Device-Bound Passkeys

The L3 **BE/BS flags** ([1] §6.1.3) are the only signal an RP gets: BE=1 marks a multi-device (synced) credential; BS marks current backup state. These bits are inside the signed authenticator data, so they are reliable *statements by the authenticator*, but nothing independently attests them for synced passkeys — see below.

- **iCloud Keychain (Apple):** passkeys sync end-to-end encrypted via iCloud Keychain; recovery via iCloud Keychain escrow guarded by device passcode (Apple platform security documentation [13][14]). Apple's position, stated by Apple engineers on the developer forums: passkeys on Apple platforms **do not provide an attestation statement**, because attestation "wasn't designed with syncing credentials in mind" and cannot meaningfully attest security properties when credentials sync to new devices [15]. `signCount` is always 0 on Apple platform credentials **[unverified against current OS]**, removing clone detection.
- **Google Password Manager:** passkeys end-to-end encrypted in sync; on new devices decryption is gated by the GPM PIN or an enrolled device's screen lock [16]. Per FIDO-dev discussion of Google's implementation, synced GPM passkeys likewise return no meaningful (i.e., `none`) attestation; the thread's framing: attestation for a multi-device credential would represent "the sync fabric", not a hardware boundary [17].
- **Device-bound:** FIDO2 security keys (and Windows Hello TPM credentials, `tpm` attestation format, [1] §8.3) provide real attestation chains; the AAGUID keys into the **FIDO Metadata Service** for certified security levels. Enterprise attestation can identify individual devices in managed deployments.

**Implication for "sole control"** (the eIDAS-style requirement that signature-creation data be usable only under the signatory's control, cf. Stream 4): for synced passkeys, the private key (a) exists on every synced device, (b) transits a provider-operated, provider-designed escrow/recovery system gated by knowledge factors (device passcode / GPM PIN), and (c) is *unattested* — the RP cannot cryptographically distinguish a genuine iCloud passkey from a software emulation that sets BE=1. A passkey-native signature scheme therefore cannot prove sole control or key provenance for the default consumer passkey; only device-bound, attested authenticators (security keys, some enterprise configurations) can. This is the single strongest contradiction of "passkeys can be qualified-signature-grade" assumptions.

## 6. Recovery & Cross-Device

- **Hybrid transport (formerly caBLE):** cross-device authentication where a phone-resident passkey signs for a session on another machine: QR code carries key material, BLE advertisement proves physical proximity, and the actual CTAP traffic flows through a vendor tunnel service. Defined in FIDO CTAP 2.2 (hybrid transports section) [10] **[section number not verified this session]**. The RP sees only a normal assertion (transport `hybrid` may be reported); it cannot distinguish or veto tunnel-mediated signing beyond transport hints.
- **Recovery:** there is no in-protocol credential recovery. Synced passkeys are recovered by recovering the *cloud account* (iCloud Keychain escrow: passcode-gated HSM escrow with recovery flows including SMS + passcode [14]; Google account recovery + GPM PIN [16]). The RP receives **no signal** that a recovery occurred — the same public key simply continues to work from a new device. `signCount` ([1] §6.1.1) was designed for clone detection but platform authenticators commonly return 0, and the spec makes RP enforcement optional. For a signing service, this means the entire account-recovery attack surface of Apple/Google accounts sits silently underneath every "signature". Yubico's earlier ARKG-based *recovery extension* proposal (backup security keys registered without being present) was never adopted into the spec [18] — its ARKG core resurfaced in [5].

## 7. Platform Limitations — what an RP can never observe or control

1. **No raw key access, no sign-this-buffer API.** The only signing primitive exposed is the assertion over `authData‖SHA-256(clientDataJSON)`.
2. **No control of UI.** The RP cannot put a document title, hash, or any text into the passkey prompt (SPC being the narrow, payments-shaped exception).
3. **No visibility of the challenge to the user** — and no way for the user to verify what they authorize.
4. **clientDataJSON is client-authored.** The RP cannot add members; unknown-member behavior is client-defined.
5. **Attestation only at registration** (practically), and **none at all** from mainstream synced passkeys [15][17].
6. **Cannot force device-bound credentials.** There is no "must not sync" parameter; `authenticatorAttachment` and hints steer, not bind. On Apple platforms every platform credential is a synced passkey.
7. **Cannot observe sync, sharing (passkeys are user-shareable via AirDrop on Apple platforms **[unverified current behavior]**), or account recovery** — only the BS bit, self-reported.
8. **Cannot enumerate credentials** or verify continued existence; L3's `signal*` methods ([1] §5.1.10) let the RP *push* state, not read it.
9. **UV is anonymous and local**: a UV=1 bit proves an enrolled unlock, not *which* human — anyone enrolled on the device (e.g., added fingerprint) verifies. No identity proofing.
10. **Origin/RP-ID binding**: credentials are unusable outside the registered domain (Related Origins in L3 relaxes this only across the RP's own listed origins), complicating any multi-party signing ceremony hosted across vendors.
11. **Algorithm fixed at registration**; no RP-side agility afterward.
12. **Counters and flags make signatures ceremony-unique**, so a "signature" cannot be recomputed or restructured later; evidence must be archived verbatim.

## 8. Limitations Register

1. **L-1 (payload):** Assertions sign `authenticatorData || SHA-256(clientDataJSON)`, never RP-chosen bytes directly ([1] §6.3.3). Document hashes can only ride inside the challenge, doubly wrapped (base64url→JSON→SHA-256).
2. **L-2 (semantics):** Signed `type` is always `webauthn.get`/`webauthn.create` — the cryptographic statement is "authentication", not "signature" ([1] §5.8.1).
3. **L-3 (WYSIWYS):** No user-visible representation of signed content; every login is an indistinguishable potential blind signature.
4. **L-4 (attestation gap):** Synced passkeys from Apple/Google provide no attestation [15][17]; RP cannot prove key storage properties.
5. **L-5 (sole control):** Sync + provider escrow + knowledge-factor recovery defeat sole-control and non-exportability arguments for consumer passkeys [13][14][16].
6. **L-6 (key entanglement):** Same key for login and any challenge-stuffed "signature"; weakens non-repudiation.
7. **L-7 (no transaction display):** txAuthSimple removed in L2 for non-implementation [7][8]; no authenticator-display path exists.
8. **L-8 (evidence format):** The assertion envelope is alien to CMS/CAdES/JAdES/COSE_Sign1 verifiers; bespoke archival format required ([6]; Stream 3 dependency).
9. **L-9 (algorithms):** ES256 is the only universal algorithm; no RP-side algorithm agility post-registration; no PQC.
10. **L-10 (recovery opacity):** Account-recovery events are invisible; signCount unreliable on platform authenticators ([1] §6.1.1).
11. **L-11 (sign extension immaturity):** PR #2078 has no implementations beyond Yubico internal PoCs, no WG commitments, L4/~2028+ timeline [3].
12. **L-12 (identity gap):** UV proves local unlock, not legal identity; binding a passkey to a natural person requires an external identity layer (Stream 2).

## 9. Opportunities for Document Signing — three paths

**(a) Direct assertion over a document hash via `challenge` — "challenge stuffing." Feasible today; weakest evidence.**
Works in every browser/authenticator with no changes; vendor-documented [6]. UV bit inside the signed bytes is genuinely valuable (proof of PIN/biometric at signing moment). But it inherits L-1…L-6, L-8: the signature is semantically an authentication, invisible to the user, made with an unattested and possibly synced key, in a nonstandard envelope. Verdict: acceptable only for **basic/advanced-lite electronic signature** claims where the audit trail (not the cryptography) carries the legal weight — essentially what incumbent e-sign vendors already do with any auth factor. A PassSign profile could harden it (dedicated signing-only RP ID and credential so the key is never used for login, addressing L-6; a signed "document manifest" as the challenge preimage; mandatory UV; archived clientDataJSON/authData). It cannot be hardened past L-3 (no WYSIWYS) or L-4/L-5 (synced-key attestation/sole control). An SPC-style browser-rendered ceremony [12] is the only existing mechanism that fixes L-3, but SPC's displayed fields are payments-specific and its availability is Chromium-bound.

**(b) The WebAuthn `sign` extension (PR #2078). Architecturally correct; not available this decade for products.**
Signs raw input, separates signing keys from authentication keys, contemplates attestation of hardware-bound signing keys, and is explicitly motivated by EUDI-wallet-grade requirements [3][4]. Combined with ARKG [5], it supports one-passkey→many-signing-keys without extra ceremonies. This resolves L-1, L-2, L-6, and (with attested security keys) L-4; it does not by itself resolve L-3 (display) or L-5 for synced implementations. Risks: individual-draft/PR status, one-vendor momentum, Apple exploring a different shape (WebKit remote-cryptokeys), L4 FPWD ~Q4 2028. Verdict: **track and shape** (participation in the WG/repo is cheap and the PassSign use case is exactly the one named in the PR), but do not build product plans on it before ~2029–2030.

**(c) Passkey-authorized separate signing key — the composable near-term architecture.**
Keep WebAuthn as the *authorization ceremony* and hold the actual signing key elsewhere; sign in a standard format (CAdES/PAdES/JAdES/COSE) with that key. Variants: (c1) **server/HSM-held key per user**, activated by a fresh UV=1 passkey assertion whose challenge binds the document manifest — this is precisely the eIDAS **remote-QSCD** pattern (Stream 4), with the passkey as the sole-control activation factor; (c2) **client-derived key via the `prf` extension** — deterministic, offline-capable, but the WG itself concluded such keys are extractable by the client process ([3], re #1945), so c2 caps out below hardware-grade; (c3) **largeBlob-carried certificates** binding identity to the credential ([1] §10.1.5), support-limited. Verdict: **(c1) is the recommended PassSign baseline** — it is the only path that simultaneously yields standards-conformant signature objects, real sole-control arguments (HSM + fresh UV assertion), identity binding via certificates, and works with today's synced passkeys (whose attestation gap stops mattering because they are an authentication factor again, not the signature key). Its costs: a trusted service operator, and the WYSIWYS problem moves to the RP's web UI (as it is for all remote signing today).

**Bottom line:** WebAuthn as deployed is an authentication protocol whose signed payload structurally resists reuse as a document-signature primitive; the standards path that fixes this (sign extension) is real but ~3–4 years out; the composable path (passkey-activated remote signing key) is buildable now from existing standards and matches the remote-QSCD pattern regulators already accept.

*This analysis is technical research, not legal advice; regulatory conclusions (eIDAS classifications, sole-control sufficiency) require review by qualified counsel and are covered by Stream 4.*

## Sources

1. W3C, *Web Authentication: An API for accessing Public Key Credentials — Level 3* (TR). https://www.w3.org/TR/webauthn-3/ — §§5.8.1, 5.8.5, 5.8.6, 6.1, 6.1.1, 6.1.3, 6.3.2, 6.3.3, 6.5, 6.5.1, 6.5.3, 6.5.5, 7.1, 7.2, 10.1.4, 10.1.5, 10.2, 13.4.3, 13.4.4, 16.x. (Section numbering verified against live TOC 2026-07-17; page too large to fetch in full.)
2. W3C, *Web Authentication Level 2* (Recommendation). https://www.w3.org/TR/webauthn-2/
3. w3c/webauthn PR #2078, "Add 'sign' extension" (emlun), incl. comments by OR13, selfissued, ve7jtb, marsouin, nsatragno. https://github.com/w3c/webauthn/pull/2078 (fetched 2026-07-17)
4. YubicoLabs/webauthn-sign-extension (iteration repo; editor's copy). https://github.com/YubicoLabs/webauthn-sign-extension ; https://yubicolabs.github.io/webauthn-sign-extension/#sctn-sign-extension
5. IETF, *The Asynchronous Remote Key Generation (ARKG) algorithm*, draft-bradleylundberg-cfrg-arkg-10 (active individual I-D, updated 2026-02-27, expires 2026-08-31). https://datatracker.ietf.org/doc/draft-bradleylundberg-cfrg-arkg/ (fetched 2026-07-17)
6. Yubico Developers, *Using WebAuthn for Signing*. https://developers.yubico.com/WebAuthn/Concepts/Using_WebAuthn_for_Signing.html (fetched 2026-07-17)
7. w3c/webauthn issue #1386, "Remove unimplemented extensions". https://github.com/w3c/webauthn/issues/1386
8. Mozilla Bugzilla 1540778, "Implement WebAuthn Simple Transaction Authorization Extension (txAuthSimple)" (never implemented). https://bugzilla.mozilla.org/show_bug.cgi?id=1540778
9. w3c/webauthn issue #2022, "Revised txAuthSimple extension". https://github.com/w3c/webauthn/issues/2022
10. FIDO Alliance, *Client to Authenticator Protocol (CTAP) 2.x* specifications. https://fidoalliance.org/specs/ (hybrid transport, hmac-secret, largeBlobs, credProtect; not fetched in full this session)
11. IANA, *COSE Algorithms* registry (ES256 −7, EdDSA −8, ES384 −35, ES512 −36, RS256 −257; RS256 registered by RFC 8812 for WebAuthn). https://www.iana.org/assignments/cose/cose.xhtml
12. W3C, *Secure Payment Confirmation*. https://www.w3.org/TR/secure-payment-confirmation/
13. Apple Support, *iCloud Keychain security overview*. https://support.apple.com/guide/security/icloud-keychain-security-overview-sec1c89c6f3b/web
14. Apple Support, *Secure keychain syncing / escrow*. https://support.apple.com/guide/security/secure-keychain-syncing-sec0a319b35f/web
15. Apple Developer Forums (Apple staff response), passkeys and attestation, thread 710302. https://developer.apple.com/forums/thread/710302
16. Google, passkeys in Google Password Manager — supported environments & GPM PIN. https://developers.google.com/identity/passkeys/supported-environments
17. FIDO-dev list, "Details on Google's implementation of passkeys". https://groups.google.com/a/fidoalliance.org/g/fido-dev/c/nhpxExcofb8
18. Yubico, *webauthn-recovery-extension* (ARKG precursor). https://github.com/Yubico/webauthn-recovery-extension
19. W3C WebAuthn WG charter (current & drafts; 2026–2028 draft scope item "Signing additional data" per parallel stream — fetch of draft timed out this session). https://www.w3.org/groups/wg/webauthn/ ; https://github.com/w3c/charter-drafts


---

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


---

# Stream 3: Digital Signature Standards — How Documents Are Signed Today

**PassSign research program — Stream 3 of 6**
**Date:** 2026-07-17
**Scope:** PDF/CMS/AdES signature anatomy; feasibility analysis of packaging WebAuthn (passkey) assertions into standard signature containers; prior art; X.509 issuance for passkeys; timestamping/LTV; verification ecosystem; remote-signing standards (CSC/ETSI/CEN).
**Method note:** Normative claims are cited to primary specifications where possible. Claims that could not be verified against a fetched primary source in this run are marked **[unverified]**. This document builds on a confirmed prior finding: GitHub searches found **no dedicated WebAuthn+CMS/AdES integration projects**, and the community-curated awesome-webauthn list contains no document-signing/AdES projects [19].

---

## 1. PDF Signature Anatomy (ISO 32000-2 §12.8)

A digitally signed PDF embeds the signature inside the document itself, using four interlocking mechanisms:

**Signature dictionary and ByteRange.** A signature field's value is a signature dictionary containing (among others) `/Filter` (handler, conventionally `Adobe.PPKLite`), `/SubFilter` (encoding of the signature blob), `/Contents` (the signature blob itself as a hexadecimal string), and `/ByteRange` — an array of four integers `[offset1 length1 offset2 length2]` defining two spans of the file's exact bytes that are covered by the signature. The two spans are everything **except** the `/Contents` hex string itself: the signature cannot cover its own value, so a "hole" is left for it. The signing software computes the digest over the ByteRange bytes, produces the signature container, and writes it into the pre-allocated (zero-padded) `/Contents` gap. Consequence: the signed object is the *serialized file bytes*, and any byte outside the hole is tamper-evident. (ISO 32000-2 §12.8.1; ISO 32000-2 is paywalled — anatomy corroborated by PDF industry documentation [2][3].) **[unverified: exact ISO clause wording; the structure itself is uncontroversial and implemented identically by Adobe, Apryse, Nutrient, iText, open-source pdfbox, etc.]**

**SubFilter values.** Legacy Adobe signatures use `/adbe.pkcs7.detached` (detached CMS over ByteRange bytes) or `/adbe.pkcs7.sha1`. PAdES signatures use `/ETSI.CAdES.detached` (ETSI EN 319 142-1), and document timestamps use `/ETSI.RFC3161` [3][6]. The important precedent: **new SubFilter values have been added to the PDF ecosystem before** (ETSI added two), so the container has an extension point — but every new SubFilter required standardization plus multi-year validator rollout.

**Incremental updates.** PDF supports appending revisions after the `%%EOF` without touching earlier bytes. A second signature, a document timestamp, or LTV data is added as an incremental update; each signature's ByteRange covers the file *up to its own revision*. This is how serial multi-party signing and post-signature augmentation (LTV, archival timestamps) work without invalidating earlier signatures.

**DocMDP / certification signatures.** The first ("certification") signature may include a `/Reference` array with a DocMDP transform whose `/P` parameter constrains permitted post-signing changes: 1 = no changes, 2 = form fill-in and signing, 3 = plus annotations. Approval signatures (ordinary signatures) do not carry DocMDP. Validators check that incremental updates after a certified revision stay within the declared permission level.

**What exactly is embedded:** the DER-encoded CMS `SignedData` structure sits inside `/Contents` — signer certificate(s) and chain, signed and unsigned attributes, the signature value, optionally an embedded RFC 3161 timestamp token, and (for LTV) revocation material either inside the CMS or in the document-level `/DSS` dictionary (Section 6). A signed PDF is therefore a *self-contained verification package*: document bytes + CMS + certificates + timestamps + revocation data.

**Design implication for PassSign:** the PDF layer is agnostic about what is inside `/Contents`; the byte-coverage model only requires that the signature scheme ultimately bind to a digest of the ByteRange bytes. The constraint lives one layer down, in CMS.

---

## 2. CMS / PKCS#7 SignerInfo: The Byte Model (RFC 5652)

CMS `SignedData` (RFC 5652 §5) contains `encapContentInfo` (absent/detached for PDF), a certificate bag, and one or more `SignerInfo` structures (§5.3) with: `sid` (issuer+serial or subjectKeyIdentifier identifying the signer's **X.509 certificate**), `digestAlgorithm`, optional `signedAttrs`, `signatureAlgorithm`, `signature`, optional `unsignedAttrs` [1].

**The critical byte-level fact:** when `signedAttrs` is present (mandatory in all AdES profiles), **the signature is NOT computed over the document**. Per RFC 5652 §5.4, the message-digest calculation is performed over "the complete DER encoding of the SignedAttrs value," re-tagged with an EXPLICIT `SET OF` tag instead of the `IMPLICIT [0]` tag used on the wire; the signature (§5.5) is then computed over that DER structure [1]. The document is bound only *indirectly*: the `signedAttrs` set MUST contain (§11):

- `content-type` (§11.1) — OID of the encapsulated content type;
- `message-digest` (§11.2) — the hash of the actual content (for PDF: the ByteRange bytes);
- optionally `signing-time` (§11.3) — though PAdES baseline uses the signature dictionary's `/M` entry instead and prohibits the CMS signing-time attribute **[unverified: exact PAdES clause]**;
- in AdES: `signing-certificate-v2` (ESS, RFC 5035), binding the signer's certificate hash into the signed bytes.

So the chain of trust in every standards-compliant signed document today is:

```
document bytes → SHA-2 digest → messageDigest attribute
             → DER(SignedAttributes) → SHA-2 digest → raw signature primitive (ECDSA/RSA/EdDSA)
```

**This two-stage indirection is the exact structural analogue of WebAuthn's own indirection** (challenge → clientDataJSON → hash → concatenated with authenticatorData → signature) — but the two indirection formats are byte-incompatible, which is the crux analyzed in Section 4.

---

## 3. The AdES Family: CAdES, XAdES, PAdES, JAdES

ETSI's Advanced Electronic Signature (AdES) formats are profiles layered on generic container formats, adding the attributes needed for legal-grade, long-lived signatures. All four share a common level model — **B-B** (basic: signed attributes incl. signing certificate reference), **B-T** (adds a trusted RFC 3161 timestamp on the signature), **B-LT** (adds all validation material: certificate chain + revocation data), **B-LTA** (adds periodic archival timestamps for decades-scale validity) — and a common validation procedure defined in ETSI EN 319 102-1 [8].

- **CAdES** (ETSI EN 319 122-1) profiles CMS: mandatory `signing-certificate-v2`, `signature-time-stamp` unsigned attribute for -T, archive timestamps for -LTA. Applicable to any file type [5].
- **XAdES** (EN 319 132-1) is the XML-DSig equivalent, with `QualifyingProperties`. Signing input is the canonicalized (C14N) `SignedInfo` element — XML canonicalization adds complexity irrelevant to PassSign except as a cautionary tale.
- **PAdES** (EN 319 142-1) profiles CAdES for embedding in PDF via `/ETSI.CAdES.detached`, adds PDF-specific rules (one SignerInfo only, ByteRange must cover the whole revision, `/M` for time, DSS/VRI for LT, document timestamps for LTA) [6].
- **JAdES** (ETSI TS 119 182-1) profiles **JWS (RFC 7515)**: the newest and least deployed member, targeted at JSON/API ecosystems [7].

### JAdES focus: how close is its signing input to a WebAuthn assertion?

JWS computes the signature over the **JWS Signing Input**: `ASCII(BASE64URL(UTF8(protected header)) || '.' || BASE64URL(payload))` (RFC 7515 §5.1) [4]. JAdES adds ETSI header parameters — `sigT` (claimed signing time), `x5c`/`x5t#S256` (certificate binding, the JWS-native equivalent of signing-certificate-v2), `sigD` (a detached-payload mechanism referencing external documents by URI + digest, so the effective payload can be a compact digest manifest rather than the document itself), and the unprotected `etsiU` array carrying timestamps and validation data for T/LT/LTA levels [7].

**Answer to the research question: structurally close, byte-wise incompatible.** A WebAuthn assertion signature is computed over `authenticatorData || SHA-256(clientDataJSON)` (WebAuthn L3 §6.3.3; verified in §7.2: "verify that sig is a valid signature over the binary concatenation of authData and hash") [9]. The JWS Signing Input is a string over the base64url alphabet plus `.`; `authenticatorData` is arbitrary binary (rpIdHash ‖ flags ‖ signCount ‖ extensions). The two byte strings cannot be made to coincide except by contrivance, so **no existing JWS `alg` value can validate a WebAuthn assertion as a JAdES signature**. However, JAdES is the *most amenable* container for a new profile, because:

1. JOSE has a lightweight, active registry for new `alg` values (vs. new ASN.1 AlgorithmIdentifier OIDs plus DER parameter structures for CMS);
2. `clientDataJSON` is itself JSON — carrying it in a JWS unprotected header is idiomatic, whereas in CMS it needs a new ASN.1 attribute;
3. the `sigD` detached mechanism already establishes the pattern "payload = digest manifest of external documents," which maps naturally onto "challenge = digest of signing input."

A hypothetical `"alg": "WebAuthn-ES256"` could define: challenge := BASE64URL(SHA-256(JWS Signing Input)); signature := assertion signature; unprotected header carries `authenticatorData` + `clientDataJSON`; verifier recomputes both indirection layers. This is exactly the kind of two-layer verification the ecosystem has never standardized. No such JOSE registration or ETSI work item was found **[negative finding; unverified as exhaustive]**.

---

## 4. **The Crux: WebAuthn Assertions Inside Standard Containers — Analysis & Prior Art**

### 4.1 The incompatibility, stated precisely

| | CMS/CAdES/PAdES | JWS/JAdES | WebAuthn assertion |
|---|---|---|---|
| Bytes actually signed | DER(SignedAttributes), EXPLICIT SET OF re-tag (RFC 5652 §5.4) [1] | ASCII(B64URL(header)‖'.'‖B64URL(payload)) (RFC 7515 §5.1) [4] | authenticatorData ‖ SHA-256(clientDataJSON) (WebAuthn §6.3.3) [9] |
| Document binding | messageDigest signed attribute | payload or sigD digest manifest | RP-chosen `challenge` inside clientDataJSON |
| Signer identity | X.509 cert referenced by `sid` + signing-certificate-v2 | `x5c`/`x5t#S256` | bare COSE public key, no certificate |
| Extra signed context | attributes chosen by signer | protected header | origin, type, crossOrigin (clientDataJSON); rpIdHash, UV/UP flags, signCount (authData) |

The underlying primitives are compatible — ES256/EdDSA/RS256 are all valid CMS and JWS algorithms — but an authenticator **refuses to sign caller-chosen bytes**; it always injects its own envelope. This is a deliberate security property (domain separation prevents an RP from tricking an authenticator into signing, e.g., a Bitcoin transaction or another RP's login challenge), and it is precisely what breaks container compatibility.

### 4.2 Could a CMS/AdES profile treat the assertion as SignatureValue?

Technically yes — cryptographic verifiability survives. The profile would be:

1. Build normal PAdES `SignedAttributes` (content-type, messageDigest over ByteRange, signing-certificate-v2).
2. Compute `H = SHA-256(DER-EXPLICIT(SignedAttributes))` and set `challenge = BASE64URL(H)` in the WebAuthn `get()` call.
3. Place the assertion's raw signature bytes in `SignerInfo.signature`.
4. Carry `authenticatorData` and `clientDataJSON` in new attributes (unsigned attributes suffice — they are integrity-protected transitively, because tampering with either changes the bytes the signature verifies over).
5. Define a new `signatureAlgorithm` OID, e.g. `webauthn-assertion-with-ES256`, whose verification procedure is: reconstruct `authData ‖ SHA-256(clientDataJSON)`, verify ECDSA; parse clientDataJSON; check `type == "webauthn.get"` and `challenge == BASE64URL(SHA-256(DER(SignedAttributes)))`; then proceed with standard CMS/AdES validation.

**What breaks in existing validators — everything at the last mile:**

- **Declared-algorithm mismatch.** If the profile lies and declares `ecdsa-with-SHA256`, every validator (Adobe, EU DSS, OpenSSL) computes the signature check over DER(SignedAttributes) and gets **TOTAL-FAILED** — a *red X*, worse than unknown. If it declares a new OID, validators return unknown-algorithm errors (EU DSS: INDETERMINATE; Adobe: "signature validity is unknown") [8][20][21]. There is no graceful degradation path.
- **Certificate requirement.** `sid` must reference an X.509 certificate; AdES-B additionally mandates signing-certificate-v2. A bare passkey has no certificate — Section 5's problem is a hard dependency, not an optional nicety.
- **PAdES SubFilter.** A conforming PAdES validator checks the SubFilter; a WebAuthn-CMS blob would need a new value (precedent: `/ETSI.RFC3161` was added for document timestamps), again requiring standardization + rollout.
- **EN 319 102-1 validation model.** The standardized validation process assumes X.509 chain building to a trust anchor, revocation checking, and signature verification per registered algorithms [8]. A WebAuthn profile must either map onto this model (cert for the passkey + new algorithm entry in ETSI TS 119 312 algorithm catalogue **[unverified: 119 312 process details]**) or define a parallel validation standard.

**Verdict:** an "assertion-as-SignatureValue" profile is *cryptographically sound and specifiable in ~10 pages*, but yields signatures that **zero deployed validators accept today**. Its value is as a standardization target, not an interop hack.

### 4.3 Prior art survey

- **Yubico, "Using WebAuthn for Signing" (developer concept doc)** — the canonical description of the *naïve* approach: replace the random challenge with the document hash, archive clientDataJSON + authenticatorData + signature, verify against the registration public key [11]. Explicitly notes the signature is "for an object comprising both the client data … and authenticator data," i.e., a **proprietary verification format**. No container, no certificate, no timestamp, no AdES mapping. This confirms the approach works but stops exactly where PassSign would begin.
- **W3C WebAuthn "sign" extension — PR #2078 (emlun/Yubico, open as of May 2025)** — the decisive piece of prior art [10]. It creates a **separate signing key pair** associated with (but distinct from) the credential key, which signs **"the given input unaltered"** — no authenticatorData/clientDataJSON envelope. Distinctness preserves domain separation while enabling raw signatures. Motivating use cases in the PR: "attested, hardware-bound signing keys for … digital identity wallets" and "general-purpose digital signatures, with seamless interoperability with existing cryptographic protocols." Status signals from the PR thread: Yubico has "rough internal proof-of-concept prototypes … nothing yet ready to share"; no other WG implementation commitments; design iteration moved to yubicolabs/webauthn-sign-extension; a companion `kem` extension is planned; the author engaged the IETF COSE/JOSE lists [10][12]. EUDI-wallet actors (ve7jtb) see it as a candidate WSCD interface for wallets.
  **Key insight: even if #2078 ships, PassSign-style container/profile work is STILL needed.** The extension solves only the byte-format problem (the authenticator can now sign DER(SignedAttributes) or a JWS Signing Input directly, producing a signature standard validators can verify mechanically). It does **not** provide: (a) an X.509 certificate binding the sign-extension public key to a legal identity; (b) an AdES profile saying which attributes/headers to use and how attestation is conveyed; (c) trust-list acceptance; (d) timestamping/LTV integration; (e) any normative statement that a UV-gated hardware signature meets AdES/QES "sole control" requirements. The extension is an enabler, not a solution.
- **Yubico/webauthn-sign-kem-extensions repo** — earlier draft of the same extensions; superseded by PR #2078 [12].
- **FIDO Alliance × eIDAS white papers (2020, 2023)** — FIDO's own position is **not** "sign documents with passkeys" but "use FIDO2 as the *authentication* layer toward a QTSP that holds the signing key server-side" ([13][14]; see Section 8). This is the strongest signal that industry consensus routed around the container problem rather than solving it.
- **Academic:** arXiv 2601.06554 (Jan 2026, Bicakci et al.) is the *inverse* problem — using QES-grade PKCS#11 hardware to back a virtual FIDO2 authenticator — confirming the gap is recognized but attacked from the opposite direction [16]. No paper was found that packages WebAuthn assertions into CAdES/PAdES/JAdES **[negative finding; unverified as exhaustive]**.
- **GitHub / community:** prior-run finding confirmed — no dedicated WebAuthn+AdES projects; awesome-webauthn lists none [19]. The OR13 comment on PR #2078 describes ad-hoc "overloading the challenge to sign arbitrary data … it's all gross" for wallet credential binding [10] — practitioners hack around the gap; nobody has standardized through it.
- **ETSI/W3C formal work items:** no ETSI ESI work item on FIDO/WebAuthn signature formats was found **[unverified as exhaustive; ETSI work programme not directly searchable in this run]**.

**Conclusion of crux:** the composition gap is real, recognized in fragments across four communities (W3C, FIDO, ETSI, academia), and unclaimed as a unified standardization effort.

---

## 5. X.509 / CA Model: Certifying a Passkey Public Key

Every AdES signature requires the signer's X.509 certificate. Three issuance paths for a passkey public key:

**(a) Classic CSR — blocked.** PKCS#10 (RFC 2986) proof-of-possession requires the requester to sign the DER-encoded `CertificationRequestInfo` with the subject private key [17]. A WebAuthn credential *cannot* produce that signature: the authenticator will only emit `Sign(authData ‖ SHA-256(clientDataJSON))`, never a raw signature over caller-supplied DER. Embedding the CSR hash in the challenge produces a signature that is *not* a valid PKCS#10 self-signature, so standard CA software rejects it. (With the #2078 sign extension, raw CSR signing becomes possible — another reason the extension matters.)

**(b) Attestation-based issuance — viable today, requires CA policy work.** At credential creation, the authenticator can return an **attestation statement**: a signature by an authenticator-model key (chaining to the vendor and verifiable against FIDO MDS) over data that includes the newly created credential public key [9]. A CA could accept this as key-binding evidence in lieu of PKCS#10 PoP — the attestation proves the key was generated on, and is bound to, a certified authenticator, which is arguably *stronger* than PoP. Precedent exists: ACME `device-attest-01` (RFC 9720-track draft; used by Apple Managed Device Attestation) issues certificates based on device attestations **[unverified: exact ACME draft/RFC status]**. Caveats: consumer platform passkeys frequently return attestation `none` (Apple in particular suppresses it for synced passkeys) **[unverified for current platform behavior]**, so this path works best with security keys and enterprise-attestation deployments; RFC 4211 (CRMF) already admits PoP methods other than signature, giving standards cover for CA/Browser-Forum-style policy definitions.
**(c) Challenge-based issuance.** The CA runs a WebAuthn ceremony itself (challenge = CA-chosen nonce, optionally binding the CSR content hash), verifies the assertion with the claimed public key, and treats it as PoP. Cryptographically sound; not describable in existing CA protocol standards (EST/CMP/ACME) without a new PoP type — a small, well-scoped standardization item.

Additional model frictions (well summarized from a PKI-practitioner perspective in [18]): passkeys are per-RP by design (privacy), so a certified passkey becomes a cross-context identity — reintroducing exactly the linkability WebAuthn avoids; no revocation infrastructure exists on the passkey side; synced passkeys (private key material replicated via cloud keychains) may fail "sole control" requirements for higher assurance levels — a Stream 4 legal question, flagged here.

---

## 6. Timestamping and Long-Term Validation

- **RFC 3161:** a TSA returns a `TimeStampToken` — itself a CMS SignedData over `TSTInfo` containing the requester's `MessageImprint` (hash) and genTime [15 n/a; RFC 3161]. In CAdES/PAdES-T, the imprint is the hash of the `SignerInfo.signature` value, proving the signature existed at time T. **Note for PassSign: RFC 3161 is format-agnostic — it timestamps a hash. A WebAuthn assertion signature can be timestamped today with zero changes.** This component composes cleanly.
- **LTV / DSS in PDF:** PAdES B-LT stores the full chain, OCSP responses and CRLs in the document-level `/DSS` dictionary (with per-signature `/VRI` entries), added by incremental update, so validation succeeds after certificate expiry using data captured at signing time [6]. B-LTA adds `/DocTimeStamp` (SubFilter `/ETSI.RFC3161`) document timestamps re-applied before algorithm/TSA-cert obsolescence, chaining evidence for decades.
- **PassSign gap:** LTV is built entirely around X.509 revocation artifacts. A passkey-native scheme must define what "revocation data" means (credential still on RP allow-list? attestation chain status? nothing?). If the passkey key is certified (Section 5), standard LTV applies unchanged — another argument for the certificate route.

---

## 7. Verification Ecosystem: What "Validates in Existing Tooling" Requires

- **Adobe Acrobat/Reader** validates against the **AATL** (Adobe Approved Trust List) and EUTL; green-check requires: chain to an AATL/EUTL root, valid revocation status, intact ByteRange digest, and a signature that verifies under a *recognized* AlgorithmIdentifier [21]. Anything else renders "validity unknown" or "invalid" — and in practice, a yellow/red badge is a document that counterparties reject.
- **EU DSS library** (EC Digital Building Blocks, open source) is the reference implementation of EN 319 102-1 validation and the engine behind many national validators; it emits TOTAL-PASSED / INDETERMINATE / TOTAL-FAILED with sub-indications [8][20]. Nonstandard algorithm OIDs or missing certificates yield INDETERMINATE at best.
- **Fate of nonstandard schemes:** DocuSign-style platform "signatures" historically wrap a *platform* certificate (the vendor signs, the user merely clicks), which validates in Adobe because the vendor cert is AATL-listed — the user's own key never signs. Proprietary formats that skip CMS entirely (e.g., naive WebAuthn archives per [11]) are invisible to the entire validation ecosystem and carry evidentiary weight only via the operator's audit logs. History (XML-DSig custom profiles, national one-off formats) shows schemes that don't validate in Adobe/DSS get relegated to closed ecosystems.
- **Bar for PassSign:** either (i) produce byte-perfect PAdES/CAdES via a certified key (server-held or #2078-extension), which validates everywhere immediately, or (ii) standardize a new profile through ETSI + get validator implementations (DSS first — it is open source and the realistic beachhead) — a multi-year path.

---

## 8. Remote Signing: CSC, ETSI TS 119 432, CEN EN 419 241-2 — Where Passkeys Already Fit

The remote-signing stack is today's *only* production-grade junction of passkeys and standard signatures:

**CSC API v2.2 (Cloud Signature Consortium, published 6 Nov 2025)** [22][23]: the signing key lives in a QTSP/RSSP HSM. Flow: OAuth2/service auth → `credentials/list` → `credentials/info` → credential authorization → `signatures/signHash` (base64url hashes in, raw signature values out) → client assembles CAdES/PAdES locally. Credential authorization is either explicit **SAD** (Signature Activation Data, from `credentials/authorize`) or, new in v2.x, an **OAuth2 access token with `credential` scope**, which "can be used instead of a classical SAD" and can be hash-bound so the authorization cannot be replayed to sign different content [23]. Because authorization rides on OAuth2, **a passkey can already be the user-facing authentication/authorization factor** at the RSSP's authorization server today — with zero changes to any signature format. The output is perfect PAdES that Adobe validates.

**ETSI TS 119 432 V1.3.1 (2026-03)** standardizes protocols for remote signature creation (XML and JSON bindings), aligned with the CSC API; the EC has assessed v1.2.1/1.3.1 and an EN is planned for ~2027-06 [24][25]. **CEN EN 419 241-2** is the Common Criteria protection profile for the QSCD server-signing system: a certified **Signature Activation Module (SAM)** inside the HSM boundary enforces **SCAL2** ("sole control assurance level 2") — the SAD must cryptographically bind (signer identity, signing key, hash of the data to be signed), so not even the QTSP's operators can activate the key without the signer's fresh authorization [26] **[unverified: exact EN 419 241-2 clause; corroborated by Entrust SAM service description and Thales/Securosys documentation]**.

**Where WebAuthn appears:** the FIDO Alliance × eIDAS white papers explicitly propose FIDO2 as the high-assurance authentication mechanism toward QTSPs for remote QES [13][14]. The natural deep integration — **derive the SAD from a WebAuthn assertion whose challenge embeds the DTBS/R hash** — gives cryptographic (not merely session-level) sole-control binding: the passkey signature itself attests "this user authorized this exact document hash." Whether any QTSP's certified SAM implements WebAuthn-derived SAD today could not be confirmed **[unverified]**; no certified deployment was found by name. This is a concrete, near-term PassSign opportunity that requires no new signature format at all.

---

## 9. Limitations Register

| # | Limitation | Severity | Layer |
|---|---|---|---|
| L1 | Authenticator signs only authData‖SHA-256(clientDataJSON); cannot sign DER(SignedAttributes)/JWS input | Blocking (until #2078) | WebAuthn |
| L2 | #2078 sign extension: draft PR, prototype-only, no WG implementation commitments, no ship date [10] | High | WebAuthn |
| L3 | No X.509 certificate for passkey keys; CSR PoP unachievable with plain credentials (RFC 2986) [17] | Blocking for AdES | PKI |
| L4 | Platform passkeys often return attestation `none`, weakening attestation-based issuance **[unverified currency]** | High | Platform |
| L5 | Synced passkeys replicate private keys across devices/cloud → "sole control" question for AdES/QES levels | High (legal, → Stream 4) | Platform/legal |
| L6 | No AlgorithmIdentifier OID / JOSE alg for assertion-format signatures; unknown algs → INDETERMINATE/FAILED in all validators | Blocking for interop | ETSI/IETF |
| L7 | No revocation model for passkey credentials → LTV semantics undefined without a certificate | Medium | AdES |
| L8 | Adobe/AATL & EUTL acceptance requires listed roots + recognized formats; multi-year onboarding | High | Ecosystem |
| L9 | Per-RP credential scoping conflicts with cross-context signing identity (privacy-by-design vs. legal identity) | Medium | Architecture |
| L10 | No ETSI/W3C joint work item on WebAuthn signature containers found **[unverified as exhaustive]** | Opportunity/risk | Standards |

---

## 10. Opportunities for PassSign

1. **Track A (ship now, fully standard output):** passkey-authorized remote signing — passkey as OAuth2/SAD factor over CSC API v2.2 toward an RSSP-held certified key. Output is byte-perfect PAdES/CAdES; validates in Adobe/DSS today. Differentiator: specify the WebAuthn-assertion-derived SAD (challenge = DTBS/R hash) as a cryptographic SCAL2 binding — apparently unspecified anywhere; a candidate contribution to ETSI TS 119 432 / CSC.
2. **Track B (standardization play):** author the missing profile — a JAdES-first "WebAuthn-ES256" algorithm (JOSE registration + ETSI TS 119 182 extension) carrying authenticatorData/clientDataJSON in `etsiU`/unprotected headers, plus a CMS twin (new OID + attributes) and a PAdES SubFilter. Section 4.2 shows the design is straightforward; the moat is process, not cryptography. Prior-art vacuum (Section 4.3) means first-mover framing is available.
3. **Track C (contingent on #2078):** if the sign extension ships, passkey authenticators emit raw signatures → only the certificate/profile/trust-list work remains. Position PassSign as the AdES profile + CA issuance policy (attestation-based, ACME device-attest-01-style) for sign-extension keys. Engaging in the PR #2078 / yubicolabs design loop now is cheap and buys influence.
4. **Certificate issuance for passkeys** is a separable, valuable work item regardless of track: define WebAuthn-assertion PoP and attestation-based PoP for ACME/EST — small specs, broad utility.
5. **RFC 3161 composes cleanly today** — any PassSign format, even proprietary-interim, should timestamp assertions immediately to accrue evidentiary value.

*This analysis addresses technical standards feasibility only; legal effect of signature formats (eIDAS qualification, sole-control interpretation, enforceability) is Stream 4's scope, and nothing here constitutes legal advice.*

---

## 11. Sources

1. RFC 5652, Cryptographic Message Syntax, §5.3–5.5, §11 — https://www.rfc-editor.org/rfc/rfc5652
2. Nutrient, "PDF digital signature standards: PAdES & CAdES" — https://www.nutrient.io/guides/web/signatures/digital-signatures/standards/
3. eID Easy, "PDF PAdES digital signatures using ETSI.CAdES.detached" — https://eideasy.com/pdf-pades-external-digital-signatures-using-etsi-cades-detached/
4. RFC 7515, JSON Web Signature, §5.1 — https://www.rfc-editor.org/rfc/rfc7515
5. ETSI EN 319 122-1 (CAdES) — https://www.etsi.org/deliver/etsi_en/319100_319199/31912201/
6. ETSI EN 319 142-1 (PAdES) — https://www.etsi.org/deliver/etsi_en/319100_319199/31914201/
7. ETSI TS 119 182-1 (JAdES) — https://www.etsi.org/deliver/etsi_ts/119100_119199/11918201/
8. ETSI EN 319 102-1 (signature creation & validation procedures) — https://www.etsi.org/deliver/etsi_en/319100_319199/31910201/
9. W3C Web Authentication Level 3, §6.3.3, §7.2 — https://www.w3.org/TR/webauthn-3/
10. w3c/webauthn PR #2078, "Add 'sign' extension" (emlun) — https://github.com/w3c/webauthn/pull/2078 (fetched; quotes verified)
11. Yubico, "Using WebAuthn for Signing" — https://developers.yubico.com/WebAuthn/Concepts/Using_WebAuthn_for_Signing.html (fetched)
12. Yubico/webauthn-sign-kem-extensions (draft) — https://github.com/Yubico/webauthn-sign-kem-extensions ; successor: https://github.com/yubicolabs/webauthn-sign-extension
13. FIDO Alliance, "Using FIDO with eIDAS Services" (2020) — https://fidoalliance.org/wp-content/uploads/2020/06/FIDO_Using-FIDO-with-eIDAS-Services-White-Paper.pdf
14. FIDO Alliance, "Deploying FIDO2 with eIDAS QTSPs / eID schemes" (2020) — https://fidoalliance.org/wp-content/uploads/2020/04/FIDO-deploying-FIDO2-eIDAS-QTSPs-eID-schemes-white-paper.pdf
15. RFC 3161, Time-Stamp Protocol — https://www.rfc-editor.org/rfc/rfc3161
16. Bicakci et al., "QES-Backed Virtual FIDO2 Authenticators," arXiv:2601.06554 (Jan 2026) — https://arxiv.org/abs/2601.06554 (fetched; abstract verified)
17. RFC 2986, PKCS #10 Certification Request Syntax — https://www.rfc-editor.org/rfc/rfc2986
18. J. Field, "WebAuthn for PKI professionals," goodsignin.com — https://www.goodsignin.com/blog/webauthn-for-pki-professionals (fetched)
19. awesome-webauthn (negative finding, prior run) — https://github.com/herrjemand/awesome-webauthn
20. EU DSS (Digital Signature Services) library — https://ec.europa.eu/digital-building-blocks/sites/display/DIGITAL/eSignature ; source: https://github.com/esig/dss
21. Adobe Approved Trust List — https://helpx.adobe.com/acrobat/kb/approved-trust-list1.html
22. Cloud Signature Consortium, CSC API V2.2 page — https://cloudsignatureconsortium.org/resources/csc-api-v2-2/
23. CSC API v2.2 specification PDF (Nov 2025) — https://cloudsignatureconsortium.org/wp-content/uploads/2025/11/csc-api.pdf
24. ETSI TS 119 432 V1.3.1 (2026-03) — https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/01.03.01_60/ts_119432v010301p.pdf
25. EUDI standards tracker, issue #68 (TS 119 432 status, EN planned 2027) — https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/68
26. Entrust, "Signature Activation Module" service description (EN 419 241-2/SCAL2 context) — https://www.entrust.com/sites/default/files/documentation/service-descriptions/signature-activation-module-sd.pdf
