# PassSign Document 5 — Protocol Proposal

**Phase 5 deliverable** · Version 0.1 · 2026-07-17 · PassSign Research Program

This document proposes the PassSign protocol architecture. It implements the Gap Analysis verdict (Document 4): PassSign is a **signing ceremony specification plus a set of small binding profiles** over existing standards — WebAuthn, OpenID4VP/Digital Credentials API, CSC API/ETSI remote signing, AdES containers, RFC 3161. Nothing herein requires new cryptography.

**Gate decisions incorporated** (agreed at the Phase 4→5 gate):

| # | Decision |
|---|---|
| D1 | Evidence record = minimal JSON signed as plain JWS; lossless JAdES mapping maintained as a normative annex |
| D2 | Same-party deployments allowed at entry tier; evidence-record verification is normative at all tiers; higher tiers require an independent Signing Service or certified activation module |
| D3 | Weak fallbacks never qualify for re-binding; re-binding requires a re-presentable identity credential (else re-enrollment); re-binding events are visible in the evidence chain |
| D4 | v0.1 specifies the multi-signer composition primitive; order is recordable, not enforced; orchestration is application-layer |
| D5 | ORP counter-signature on the manifest and signer receipt are normative; transparency log is a named optional extension with a defined slot |

**Go-conditions carried from Document 4:** baseline = passkey-activated certificate-bearing signing key with standard AdES output (§3); protocol-level substitutability of the Signing Service role (§8); legal counsel review before publication; standards engagement in parallel (§12).

---

## 1. Design goals and non-goals

**Goals.** (G1) Any signature is verifiable offline by any third party from its evidence record alone. (G2) A signature exists only if a fresh, user-verified passkey ceremony authorized exactly that document. (G3) Output documents validate in existing tooling (Adobe, EU DSS) at the baseline. (G4) Integration effort comparable to a payments API; verification implementable from the spec alone. (G5) Assurance is labeled, never implied: every evidence record states its tier and authenticator class.

**Non-goals.** No workflow orchestration engine (D4). No trusted-display hardware requirement — WYSIWYS is addressed by accountability (D5), not prevention. No new cryptographic primitives. No replacement of eIDAS trust services — QES is reached *through* QTSPs, not around them. No central PassSign registry — trust anchors are deployment policy (Document 4, Ch. 6).

## 2. Actors

| Actor | Role | Analogue |
|---|---|---|
| **Signer** | Holds a passkey; optionally holds a wallet identity credential | End user |
| **Orchestrating Relying Party (ORP)** | The application presenting documents and initiating ceremonies; counter-signs every manifest | The merchant in a payments flow |
| **Signing Service (SS)** | Custodies signing keys (HSM), verifies passkey activation before every signature, produces container signatures and evidence records | Payment processor / remote-signing RSSP |
| **Identity Issuer** | Issues the wallet credential used at enrollment (EUDI PID, mDL, Entra VC…) — external, composed as-is | — |
| **Verifier** | Any party validating a signed document + evidence record, offline | — |
| **Timestamp Authority (TSA)** | RFC 3161, used as-is | — |
| **Transparency Log** *(optional extension, D5)* | Append-only log of evidence-record hashes | Certificate Transparency log |

At the entry tier, ORP and SS may be the same party (D2); the protocol treats them as distinct roles regardless, and every conformance requirement binds to the role, not the operator.

## 3. Architecture overview

```
                        ┌────────────────────┐
                        │  Identity Issuer    │  (enrollment only)
                        └─────────┬──────────┘
                                  │ OID4VP / DC API presentation
                                  ▼
 ┌──────────┐  manifest   ┌────────────┐  activation   ┌──────────────────┐
 │   ORP    │────────────►│   Signer   │──────────────►│ Signing Service   │
 │ (app UI) │  (counter-  │ (passkey,  │  (WebAuthn    │  HSM key + cert   │
 │          │   signed)   │  browser)  │   assertion)  │  SCAL2-style check│
 └────┬─────┘             └─────┬──────┘               └───────┬──────────┘
      │                         │  signer receipt              │
      │                         ◄──────────────────────────────┤
      │   signed PDF (PAdES) + evidence record (JWS)           │
      ◄────────────────────────────────────────────────────────┘
                                                        │ RFC 3161
                                                        ▼
                                                  ┌───────────┐
                                                  │    TSA    │
                                                  └───────────┘
```

**The baseline conformance class (Class B).** The signer's passkey never signs the document. It signs a WebAuthn assertion whose challenge is the hash of the counter-signed manifest; the SS verifies that assertion as the *activation* of a per-signer key held in its HSM (the eIDAS SCAL2 pattern); the HSM key — bearing an X.509 certificate, optionally a one-time certificate per ETSI TS 119 431-1 — produces a byte-standard PAdES/CAdES/JAdES signature. Existing validators verify the document; PassSign-aware verifiers additionally verify the evidence record, which is what makes the signature a *PassSign* signature (D2: no verifiable record, no PassSign signature).

**Conformance classes** (primitive-agnostic ceremony, per Gate 1):

- **Class A — Assertion evidence.** No SS key; the hardened challenge-stuffing profile (dedicated signing-only credential, mandatory UV, verbatim archival). Evidence-record-only output; no standard container. Entry-tier ceilings apply.
- **Class B — Authorized signing key.** The baseline above. Default.
- **Class C — Native sign extension.** When WebAuthn `sign` (w3c/webauthn PR #2078) ships: the derived key signs the container's signing input directly; certificate and evidence machinery unchanged. Future.

All three classes share one ceremony, one manifest, one evidence-record schema. A verifier learns the class from the record.

## 4. Objects and byte structures

### 4.1 Signing Manifest

A JSON object constructed by the ORP, containing at minimum: `doc` (array: content hash(es) — SHA-256 over the exact byte ranges to be signed, per document state for multi-signer), `display` (the human-readable representation shown to the signer: title, parties, summary, consent text, locale), `tier` and `class` claimed, `orp` (identifier + key reference), `signer_ref` (SS account reference, pseudonymous), `order` (optional declared position, e.g. "2 of 3" — recorded, not enforced, D4), `policy` (timestamping, log usage), `nonce`, `iat`.

The ORP signs the manifest as a JWS (**ORP counter-signature**, D5). The RP is thereby cryptographically committed to what it claimed the signer was shown, before the signer's key ever engages. Manifest schema fields each have a defined JAdES home (D1 annex): `display` → JAdES `sigPSt`/claimed attributes; `doc` → `sigD` digest manifest; timestamps → `sigTst`, etc.

### 4.2 Activation ceremony bytes

`challenge = BASE64URL(SHA-256(manifest_JWS))` in a WebAuthn `get()` with `userVerification: "required"`. The signed assertion therefore covers, transitively: the manifest (including the display claim, document hashes, and the ORP's counter-signature) and the origin of the ceremony. Assertion verification recomputes this chain. Credential scoping: the passkey is registered to the SS's RP ID; when ORP ≠ SS, the ceremony runs in a cross-origin embedded context (permission-policy-gated `publickey-credentials-get`, the SPC-established pattern) or via redirect. Same-party deployments are the degenerate same-origin case.

### 4.3 Evidence Record (D1)

A JSON structure signed as a JWS by the SS. Contents:

| Field group | Contents |
|---|---|
| `manifest` | The complete manifest JWS (embeds ORP counter-signature) |
| `assertion` | `authenticatorData` (b64u), `clientDataJSON` (b64u, verbatim), `signature`, `credentialId`, COSE public key, authenticator class (synced / device-bound / attestation present), UV flag echo |
| `identity` | Snapshot or reference of the enrollment identity presentation: issuer, credential type, trust anchor, assurance context, selective-disclosure subset, status-list proof *at enrollment and at signing* (Ch. 7 snapshot rule) |
| `binding` | Account evidence-chain head: hash link to prior events (enrollment, re-bindings — D3 visibility) |
| `container` | Reference to the produced standard signature (PAdES ByteRange digest, certificate chain reference) — absent in Class A |
| `sig_service` | SS identifier, key reference, tier attestation, activation-check statement |
| `timestamps` | RFC 3161 token(s) over the record hash |
| `log` *(optional)* | Inclusion proof from a transparency log (D5 extension slot) |

**Signer receipt (normative, D5):** the complete evidence record is delivered to the signer's agent at ceremony completion; the SS MUST NOT be the sole holder.

**Verification rule (normative, D2):** a PassSign verifier MUST validate: (1) evidence-record JWS against the SS key; (2) ORP counter-signature; (3) WebAuthn assertion against the enrolled credential key, including challenge ↔ manifest-hash equality and `type == "webauthn.get"`; (4) document hash ↔ container ByteRange digest equality; (5) timestamps; (6) status snapshots; (7) tier/class consistency (e.g., a Tier-2 record with a synced-passkey authenticator class MUST fail). All steps run offline from the record plus public material.

## 5. Ceremonies

### 5.1 Enrollment (identity ↔ passkey ↔ signing key binding)

1. Signer presents a wallet identity credential to the SS via OpenID4VP / Digital Credentials API (or, at Tier 1, a deployment-accepted alternative — labeled as such).
2. SS validates the presentation against its configured trust anchors; records issuer, type, anchor, assurance context.
3. Signer registers a passkey at the SS RP ID (`userVerification: required`; attestation captured when offered — determines eligible tiers).
4. Class B: SS provisions an HSM key pair for the account; obtains an X.509 certificate for it (per-ceremony one-time certificates permitted per ETSI TS 119 431-1).
5. An enrollment event opens the account's **evidence chain** (hash-linked event log; D3).

### 5.2 Signing

1. ORP constructs and counter-signs the manifest; submits to SS.
2. SS validates manifest schema and ORP signature; issues the WebAuthn `get()` with the manifest-hash challenge, in the signer's ceremony context (the ORP renders the document; the manifest `display` is what the ORP is committed to having rendered).
3. Signer completes user verification (biometric/PIN) — one gesture.
4. SS performs the **activation check**: assertion verifies against the enrolled credential; challenge matches; UV set; account active; tier constraints met. At Tier 2+ this check runs in an independent SS or a certified activation module (D2).
5. Class B: HSM signs the container (standard PAdES SignedAttributes flow); Class A: the assertion itself is the terminal cryptographic act.
6. SS assembles the evidence record, obtains RFC 3161 timestamp(s), appends to the account evidence chain, delivers the signed document + record to the ORP and the **receipt to the signer**, and (if policy) submits the record hash to a transparency log.

### 5.3 Verification

Exactly §4.3's normative rule. Two-stage by design: *legacy stage* — the signed PDF validates in any existing tool (Class B); *PassSign stage* — the evidence record proves who authorized it, what they were shown, with what identity assurance, on what class of authenticator. Verifiers MUST treat legacy-stage success alone as "not a PassSign signature" (D2).

### 5.4 Re-binding (D3)

Trigger: signer presents a new passkey for an existing account. Requirements: (1) fresh identity presentation whose disclosed attributes match the enrolled identity per SS policy — weak factors (email/SMS/recovery codes) are non-conforming, full stop; (2) planned migration alternative: a cross-signing ceremony where the old passkey authorizes the new one; (3) the re-binding event is appended to the evidence chain and surfaces in the `binding` group of every subsequent evidence record — verifiers can see that, and when, the activation credential changed; (4) an account without a re-presentable identity credential cannot re-bind; it re-enrolls as a new account (old signatures unaffected — their evidence is already snapshotted).

### 5.5 Multi-signer composition (D4)

Each signer's ceremony binds to the document state it actually signed (`doc` hashes per state; PDF incremental updates carry the container side). Later signatures transitively cover earlier ones; order is provable post-hoc from coverage plus timestamps. The manifest's `order` field records declared intent so evidence shows "signed as step 2 of 3 as represented." The protocol does not enforce sequence; routing, notification, and expiry are application concerns.

## 6. Assurance tiers

| Tier | Authenticator | SS requirement | Identity | Legal posture (per Doc 3; not legal advice) |
|---|---|---|---|---|
| **1 — Baseline** | Synced passkeys allowed | Same-party allowed; activation check normative but self-policed | Deployment policy (labeled) | SES with best-in-class attribution evidence |
| **2 — Assured** | Device-bound with attestation | Independent SS **or** certified activation module (EN 419 241-2 pattern) | Cryptographically-bound wallet credential, substantial+ | Defensible AdES (Art 26 mapping per Doc 3 §2) |
| **3 — Qualified** | Tier 2 + | QTSP-operated, certified QSCD, qualified certificate & timestamp | LoA high | QES via existing qualification — PassSign supplies ceremony + evidence; QTSP supplies qualification |

Tier is claimed in the manifest, attested in the record, and *checkable*: the authenticator class, attestation, SS identity, and certificate type are all in the evidence, so an inflated tier claim fails verification step (7).

## 7. Revocation, timestamping, longevity

**Snapshot rule.** All status (identity-credential status-list proof, certificate OCSP/CRL) is evaluated at signing time and embedded in the record — the LTV pattern generalized. Verification never requires a live issuer or SS (G1, substitutability).

**Timestamping.** At least one RFC 3161 token over the record hash is mandatory at all tiers; qualified timestamps at Tier 3. Re-timestamping before algorithm obsolescence follows the PAdES B-LTA pattern for both the container and the record.

**Algorithm agility.** Container algorithms follow the ETSI TS 119 312 catalogue. The record's JWS uses registered JOSE algorithms. WebAuthn's practical ES256 monoculture is inherited and documented; PQC migration arrives via the container/JOSE registries and (for Class C) authenticator algorithm evolution — no PassSign-specific cryptographic surface exists to migrate.

## 8. Substitutability (go-condition 2)

1. **Standard SS interface:** the ORP↔SS API is specified as a profile of CSC API v2.x (`credentials/*`, `signHash`) plus a PassSign extension for manifest submission and evidence assembly. Any conforming SS is interchangeable.
2. **No liveness dependency:** verification is offline (snapshot rule); a defunct SS invalidates nothing already signed.
3. **Signer portability:** moving to a new SS = re-binding at the new SS with the same identity credential (D3). No key export exists or is needed — continuity of *identity*, not of key material, is what the evidence chain proves across services.
4. **No mandatory central registry:** trust anchors (accepted issuers, SS accreditation, log operators) are deployment policy referencing existing registries (EUTL, IACA sets), per Document 4 Ch. 6.

## 9. Threat model

| Threat | Actor | Mitigation | Residual |
|---|---|---|---|
| Phishing / ceremony relay | External | FIDO origin binding; credential scoped to SS RP ID | Standard WebAuthn residuals |
| Signature without user ceremony | Rogue/compromised SS (incl. same-party) | Activation check; forged output has no valid assertion → fails normative verification (D2); Tier 2+ moves the check to independent/certified module | Tier 1 forgeries pass *legacy* validation only — mitigated by D2's verification rule and tier labeling |
| Bait-and-switch display | Malicious ORP | Counter-signed manifest commits ORP to the display claim (D5); assertion covers manifest hash; mismatch between `display` and actual document content is provable by anyone opening both | Consistent lying in display+manifest is detectable post-hoc, not preventable — trusted display is an ecosystem gap (Doc 4 Ch. 2) |
| Assertion replay / cross-document reuse | ORP / network | Challenge = manifest hash; nonce; per-ceremony freshness in activation check | — |
| Evidence suppression / history rewrite | ORP or SS | Signer receipt (D5); hash-linked account evidence chain; optional transparency log slot | Log optionality (deployment policy decision, D5) |
| Fraudulent re-binding (recovery attack) | External w/ weak factor | D3: weak factors non-conforming; identity re-presentation required; events visible in subsequent records | Stolen wallet credential + coerced biometric — inherited from wallet ecosystem |
| Sync-fabric compromise (passkey escrow) | Platform-level | Tier 1 ceiling; Tier 2+ requires device-bound + attestation; authenticator class always recorded | Consumer reality documented, not solved (industry-wide) |
| Tracking / correlation | ORP/SS/Log | Per-SS credential scoping; selective disclosure at enrollment; records carry disclosed subset only; log stores hashes | GDPR/data-minimization review flagged for RFC stage |
| TSA compromise | TSA | Multiple-TSA policy option; re-timestamping | Standard PKI residual |

## 10. UX and developer-experience requirements (normative-adjacent)

**Signer:** no account creation beyond the passkey; one UV gesture per signature; identity proved once at enrollment, reused at zero marginal cost; receipt delivered automatically; re-binding is a defined, non-punitive path (with a wallet credential).

**Developer (ORP):** integration surface = construct manifest → call SS API → receive document + record (one API + one webhook); a conforming "hello world" MUST be achievable without contacting sales (conformance test vectors published with the spec); no PassSign-specific cryptography beyond JWS signing of the manifest.

**Verifier:** a complete verifier MUST be implementable from the spec + test vectors alone, with no network access.

## 11. Open issues → RFC v0.1 (Document 6)

1. Final JSON schemas (manifest, evidence record) + media types (`application/passsign-manifest+jwt`, `application/passsign-evidence+jwt`) and registration plan.
2. JAdES mapping annex — field-by-field table, losslessness proof obligations (D1).
3. CSC API profile: exact endpoint/parameter mapping and the manifest/evidence extension.
4. Cross-origin ceremony embedding: permission policy, redirect fallback, UX text requirements.
5. Evidence-chain data structure (hash-link format; pruning/archival rules).
6. Transparency log extension: log entry format, inclusion-proof carriage (D5 slot).
7. Internationalization of `display`; accessibility requirements for the ceremony.
8. Privacy/GDPR data-minimization review of record contents (org compliance + counsel).
9. Class A container-less verification profile details; Class C binding once PR #2078 stabilizes.
10. Test vectors and conformance suite scope.

## 12. Standards engagement (parallel track, go-condition 4)

W3C Community Group formation ("Passkey Document Signing"); comment on w3c/webauthn PR #2078 with PassSign's Class C requirements; IETF individual draft for the CMS/JAdES WebAuthn-context attributes (Track B enabler); CSC liaison for the SS interface profile; ETSI TC ESI engagement deferred until CG report + implementation exist (Doc 1 venue analysis).

---

*Regulatory statements describe design intent against researched frameworks and are not legal advice; counsel review precedes publication (go-condition 3).*
