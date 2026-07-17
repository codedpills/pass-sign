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
