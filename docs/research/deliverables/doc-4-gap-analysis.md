# PassSign Document 4 — Gap Analysis

**Phase 4 deliverable (synthesis of Streams 1–6 / Documents 1–3)**
Version 0.1 · 2026-07-17 · PassSign Research Program

This is the pivotal document of the research program. It consolidates all six research streams into a single gap register and renders the verdict on the framing question: *is PassSign a genuinely new protocol, or a novel composition of existing standards?*

Every chapter follows the mandated structure — **Current capability → Limitation → Opportunity** — and every opportunity is classified:

- **Compose** — existing standard used as-is
- **Profile** — existing standard, constrained/parameterized (spec work, low risk)
- **Extend** — existing standard needs a defined extension (spec + WG engagement)
- **Invent** — nothing exists (highest risk)

Source references of the form (D2 I §4) mean Document 2, Part I (Stream 1), section 4. Legal chapters inform protocol design and are not legal advice.

---

## Verdict (stated first)

**PassSign is a novel composition of existing standards.** Of the thirteen gaps analyzed, none requires new cryptography. Ten classify as Compose or Profile, two as Extend (both with active standards vehicles already in motion), and exactly one as Invent — and that one is *protocol design, not cryptographic invention*: the signing ceremony itself, the layer that gives a WebAuthn authorization the semantics, evidence structure, and workflow of a legally meaningful signature. That is precisely the layer a new open specification is *supposed* to invent. The hypothesis survives Phase 4 intact, in its architectural reading: the passkey is the authorization and evidence root of every signature, not the raw document signer.

---

## Chapter 1 — Signing payload & semantics

**Current capability.** A WebAuthn assertion signs `authenticatorData || SHA-256(clientDataJSON)` with the credential key; RP-chosen bytes enter only via `challenge`, doubly wrapped, and the signed `type` is always `"webauthn.get"` (D2 I §2). A document hash *can* be carried in the challenge ("challenge stuffing"), and this works in every deployed browser today; Yubico documents the pattern, practitioners deploy it, and its own users call it "gross" (D2 III §4.3).

**Limitation.** The cryptographic statement is structurally "I authenticated to this origin," never "I signed this document." Key entanglement (same key for login and "signing") weakens non-repudiation; every login is an indistinguishable potential blind signature; the evidence envelope is alien to every standard verifier (D2 I §8 L-1, L-2, L-6).

**Opportunity.** Three-class ceremony abstraction adopted at Gate 1: (a) hardened assertion-evidence profile (dedicated signing-only credential/RP ID, signed document manifest as challenge preimage, mandatory UV, verbatim archival); (b) native raw signing when the WebAuthn `sign` extension ships (~2028+); (c) passkey-authorized separate signing key — the baseline, available now.

**Classification: Profile** (paths a, c) + **Extend, owned by others** (path b — w3c/webauthn PR #2078; engage, don't build). ▸ *Load-bearing: yes. Blocked: no.*

## Chapter 2 — Signing intent & ceremony (WYSIWYS)

**Current capability.** Nothing in the WebAuthn stack shows the user what they are signing. The CTAP2.0 transaction-confirmation extensions (txAuthSimple) were removed in L2 for non-implementation; the only browser-rendered precedent is Secure Payment Confirmation — a chartered W3C ceremony layered on WebAuthn, but payments-specific and Chromium-bound (D2 I §§4.2–4.3). Incumbent e-signature products handle intent at the web-UI layer and record it in platform-sealed audit logs (D1 I).

**Limitation.** What-you-see-is-what-you-sign cannot be provided by the authenticator or browser today; legally meaningful *intent to sign* (UETA §9, eIDAS recital-level expectations) must be constructed and evidenced above the WebAuthn layer. This is the deepest genuine gap found in the entire program.

**Opportunity.** Define the PassSign signing ceremony: normative display requirements on the orchestrating RP, a signed document manifest (title, hash, parties, tier, consent text) whose hash is bound into the ceremony, capture of ESIGN §101(c) consent artifacts, and an evidence record third parties verify offline. SPC provides the design pattern and the W3C precedent that ceremony-over-WebAuthn specs are charterable (D1 II §2d).

**Classification: Invent (protocol, not crypto)** — this is PassSign's core contribution. ▸ *Load-bearing: yes — and it is the deliberate novelty.*

## Chapter 3 — Identity binding (person ↔ credential)

**Current capability.** WebAuthn identifies credentials, not persons (D2 I §8 L-12). The identity stack is mature and final: VC-DM 2.0 (W3C Recommendation), SD-JWT (RFC 9901), OpenID4VCI/VP (Final), ISO 18013-5 mdocs, delivered via the Digital Credentials API (W3C WD, shipping in Chromium). Wallets (EUDI PID at LoA High, Apple/Google state IDs, Entra) carry government-grade proofing (D2 II §§1–6).

**Limitation.** A passkey public key is expressible as a VC holder-binding key (`cnf` accepts any JWK) but cannot produce a standard KB-JWT or mdoc device signature — same payload constraint as Chapter 1 (D2 II §7.2). Wallet verification access is gated (Apple entitlements; EUDI RP registration), and no uniform IAL/LoA labeling exists across wallets (D2 II §8 L-7, L-8).

**Opportunity.** Two-layer design: (i) *available now* — bind an OpenID4VP identity presentation (wallet-signed, standard proof) and a passkey assertion over the same document-bound challenge into one evidence record; identity assurance from the credential, signing continuity from the passkey; (ii) *standardizable* — a "WebAuthn proof-of-possession for VC holder binding" profile at the OID4VCI proof-type extension point, standardizing what practitioners already hack. The RP-scoping asymmetry works in PassSign's favor: a fixed ceremony RP is exactly the setting where passkey-bound credentials function (D2 II §7.2e).

**Classification: Compose** (layer i) + **Profile** (layer ii). ▸ *Load-bearing: yes. Blocked: no.*

## Chapter 4 — Signature container compatibility

**Current capability.** CMS/PAdES/JAdES containers, EN 319 102-1 validation, Adobe/EU DSS tooling — all mature. An "assertion-as-SignatureValue" profile is cryptographically sound and specifiable in ~10 pages: challenge = SHA-256(DER(SignedAttributes)), assertion signature as `SignerInfo.signature`, authenticatorData/clientDataJSON as attributes, new algorithm OID with a defined verification procedure (D2 III §4.2). JAdES is the most amenable container (JSON-native, `etsiU` extension point, JOSE registry).

**Limitation.** Zero deployed validators accept it today: unknown algorithm → INDETERMINATE/unknown; mislabeled algorithm → TOTAL-FAILED; AdES hard-requires an X.509 certificate the passkey lacks; a new PAdES SubFilter needs standardization and rollout (D2 III §4.2).

**Opportunity.** Baseline path (c1) sidesteps the problem entirely — output is byte-perfect PAdES/CAdES signed by a certified key, validating in existing tooling now. The assertion-format profile (JAdES-first "WebAuthn-ES256": JOSE registration + TS 119 182 extension + CMS twin) is the standardization play, valuable as the evidence-class format and as first-mover framing in an unclaimed space (D2 III §10 Tracks A–B).

**Classification: Compose** (baseline) + **Extend** (assertion profile: IETF JOSE/COSE registration + ETSI JAdES extension). ▸ *Load-bearing: baseline yes (Compose — safe); extension is upside, not dependency.*

## Chapter 5 — PKI binding (X.509 for passkey keys)

**Current capability.** The CA model, EU Trusted Lists, qualified certificates, and ACME issuance automation all exist. ETSI TS 119 431-1 v1.3.1 permits one-time-certificate signing where a single identification event covers issuance + signing (D3 §10.4).

**Limitation.** A plain passkey cannot sign a PKCS#10 CSR (proof-of-possession requires signing caller-chosen bytes — the Chapter 1 constraint again); attestation-based issuance is the viable route, but consumer synced passkeys typically attest `none` (D2 III §§5, 9 L-3/L-4).

**Opportunity.** In the baseline, the certificate attaches to the HSM/RSSP-held signing key — solved today with existing RA/QTSP practice, including one-time certificates per ceremony. As a separable spec: define WebAuthn-assertion PoP and attestation-based PoP challenges for ACME/EST (device-attest-01 pattern) — small, broadly useful, and the enabler for Track B/C evidence classes.

**Classification: Compose** (baseline) + **Extend** (ACME/EST PoP profile). ▸ *Blocked: no.*

## Chapter 6 — Trust bootstrapping

**Current capability.** Accreditation-registry patterns are proven at scale: EU Trusted Lists (Art 22), the ARF List-of-Trusted-Entities model, mdoc IACA root distribution, AATL (D2 II §6.1; D2 III §7).

**Limitation.** Registry acceptance is slow and political (AATL/EUTL onboarding is multi-year); wallet-verifier access is permissioned; no registry exists for "PassSign signing services" or for assurance-labeling wallet credentials (D2 II §8 L-7/L-8).

**Opportunity.** Mirror, don't invent: configurable trust lists referencing existing anchors (EUTL for QTSPs, IACA sets for mdocs, issuer accreditation per deployment), with the evidence record carrying issuer + credential type + trust anchor so verifiers apply their own policy. The protocol defines the *slot*, deployments choose the registry.

**Classification: Compose.** ▸ *Blocked: no.*

## Chapter 7 — Revocation

**Current capability.** X.509 revocation (CRL/OCSP) for certified keys; Bitstring Status List (W3C REC) / OAuth Token Status List for credentials (D2 II §1.3).

**Limitation.** Passkey credentials themselves have no revocation model — no certificate, no status protocol; RP-side allow-listing is invisible to third-party verifiers. Credential status checking creates issuer phone-home with only herd privacy (D2 III §9 L-7; D2 II §8 L-10).

**Opportunity.** Baseline inherits X.509 revocation wholesale. For evidence records: revocation is evaluated *at signing time* and snapshotted into the record (status-list proof for the identity credential, certificate status for the signing key), aligning with how LTV already freezes validation material. Passkey "revocation" is an RP lifecycle event recorded in ceremony evidence, not a verifier-facing protocol.

**Classification: Compose** (+ a small **Profile** for status-snapshot carriage). ▸ *Blocked: no.*

## Chapter 8 — Timestamping

**Current capability.** RFC 3161 is format-agnostic — it timestamps a hash. A WebAuthn assertion signature can be timestamped today with zero changes; CAdES/PAdES-T integration is standard (D2 III §6).

**Limitation.** None material.

**Opportunity.** Mandate qualified (or policy-configurable) timestamps on every evidence record from day one — evidentiary value accrues immediately, even for interim formats.

**Classification: Compose.** ▸ *The cleanest component in the entire analysis.*

## Chapter 9 — Long-term validation

**Current capability.** PAdES B-LT/B-LTA with DSS/VRI dictionaries and document timestamps preserves validity for decades; EN 319 102-1 defines the validation model (D2 III §6).

**Limitation.** LTV is built entirely around X.509 artifacts. For bare-assertion evidence, "revocation data" is undefined; algorithm agility is absent from WebAuthn (ES256 near-universal, no PQC path) (D2 I §8 L-9; D2 III §6).

**Opportunity.** Baseline gets standard LTV unchanged (another argument for the certificate route). Evidence-class records adopt the same pattern: embed validation material + archival timestamps in the record; re-timestamp before algorithm obsolescence. PQC migration rides the container standards, not PassSign-specific work.

**Classification: Compose** (baseline) / **Profile** (evidence records). ▸ *Blocked: no.*

## Chapter 10 — Recovery & credential lifecycle

**Current capability.** Synced passkeys give consumer-grade recoverability via provider escrow (iCloud Keychain, Google Password Manager); cross-device (hybrid) flows work; device-bound keys give attestation but no recovery (D2 I §§5–6).

**Limitation.** Recovery events are invisible to the RP (signCount unreliable); sync defeats sole-control and key-provenance claims; the assurance/recoverability trade-off is unresolved industry-wide — Okta blocks synced passkeys for exactly this reason, and Login.gov's answer to total authenticator loss is account deletion (D2 I §8 L-5, L-10; D1 I).

**Opportunity.** The baseline dissolves the dilemma: the *signing key* lives in the HSM/RSSP (durable, certified, sole-control via activation), while the *passkey* is only the activation factor — freely synced and recoverable, because a recovered passkey re-binds through identity verification rather than inheriting signing power silently. Re-binding after recovery is a defined ceremony with fresh identity presentation; authenticator class is recorded per signature.

**Classification: Profile** (re-binding ceremony over existing components). ▸ *This chapter is where the composition architecture pays for itself.*

## Chapter 11 — Legal tier mapping

**Current capability.** SES everywhere by definition; Art 26's four AdES requirements (verified verbatim) are analyzable per authenticator class; the SCAL2 precedent establishes sole control as *exclusive activation capability*, not key custody; QES via QTSP remote signing is the dominant model; passkey ceremonies are plausibly the strongest commodity attribution evidence under UETA §9 / ECA s.7 / SG ETA s.18 (D3 throughout).

**Limitation.** No authoritative position exists on synced passkeys vs Art 26(c); no case law applies WebAuthn evidence to attribution; EUDI free QES commoditizes the EU consumer statutory-form segment (D3 limitations register).

**Opportunity.** Two assurance tiers as protocol profiles (synced = attribution/SES-grade; device-bound-attested = AdES-grade); QES on-ramp via passkey-as-SCAL2-activation; pursue formal ETSI/CEN recognition of WebAuthn UV assertions as an acceptable SCAL2 authentication mechanism — converting legal ambiguity into product tiers and a standards agenda (D3 §10).

**Classification: Compose** (tiers map onto existing law) + **Extend** (SCAL2 recognition — regulatory/standards engagement, not technology). ▸ *Blocked: no; risk is unquantified, not adverse.*

## Chapter 12 — Developer API surface

**Current capability.** Incumbent APIs are mature but metered and gated: DocuSign go-live reviews and envelope pricing, Adobe premium-auth procurement, identity at $2.40/attempt; Dropbox Sign proves demand for hours-not-weeks integration but caps at SES (D1 I). WebAuthn RP libraries are commodity open source; CSC API v2.2 gives a uniform server-side signing interface.

**Limitation.** No open, end-to-end "add signing to your app" protocol exists at any tier; assurance level and developer experience are inversely correlated across the market (D1 I failure-mode inventory §6).

**Opportunity.** Specify the protocol so that a conforming implementation is: one WebAuthn ceremony + one HTTPS evidence API + standard container output — integrable at Dropbox Sign effort with Signicat-class assurance. Open verification (any party validates evidence records offline) is the wedge no incumbent will match, because their audit-trail moat depends on not offering it.

**Classification: Invent-adjacent Profile** — the API shape is part of the ceremony spec (Chapter 2); components are all Compose. ▸ *Differentiation driver.*

## Chapter 13 — User ceremony UX

**Current capability.** Passkey UX (biometric prompt, cross-device QR) is the best-tested strong-auth UX in existence; wallet presentment UX is shipping on both mobile platforms; incumbents' signer UX is account creation + emailed links + OTP juggling, degrading further at higher tiers (video identification, certified-agent review) (D1 I; D2 II §5).

**Limitation.** The emailed capability-URL remains the industry's front door and its biggest phishing surface (vendor-acknowledged); occasional signers bear compounding friction; QES flows are heavyweight everywhere (D1 I failure-mode inventory §§8–10).

**Opportunity.** PassSign's ceremony: no account, no emailed capability link (origin-bound passkey ceremony), identity verified once and reused at zero marginal cost, one biometric gesture per signature. UX requirements enter the spec as first-class conformance criteria (research principle: security users won't adopt is unlikely to succeed).

**Classification: Compose** (UX rides existing platform ceremonies; requirements codified in the ceremony spec). ▸ *Blocked: no.*

---

## Consolidated gap register

| # | Chapter | Classification | Load-bearing? | Standards vehicle |
|---|---|---|---|---|
| 1 | Payload & semantics | Profile (+Extend via PR #2078, owned by others) | Yes | PassSign spec; W3C WebAuthn L4 |
| 2 | Intent & ceremony (WYSIWYS) | **Invent (protocol)** | Yes — core novelty | PassSign spec (W3C CG); SPC as pattern |
| 3 | Identity binding | Compose + Profile | Yes | OID4VP/DC API as-is; OID4VCI proof-type ext. |
| 4 | Container compatibility | Compose (baseline) + Extend (assertion profile) | Baseline yes | CSC/ETSI as-is; IETF JOSE/COSE + ETSI TS 119 182 |
| 5 | PKI binding | Compose + Extend (ACME PoP) | No (baseline solved) | QTSP practice; IETF ACME |
| 6 | Trust bootstrapping | Compose | No | EUTL/IACA/deployment policy |
| 7 | Revocation | Compose (+small Profile) | No | X.509 + Status List |
| 8 | Timestamping | Compose | No | RFC 3161 as-is |
| 9 | Long-term validation | Compose / Profile | No | PAdES LTV pattern |
| 10 | Recovery & lifecycle | Profile | Yes (UX-critical) | PassSign re-binding ceremony |
| 11 | Legal tier mapping | Compose + Extend (SCAL2 recognition) | Yes | ETSI/CEN engagement |
| 12 | Developer API | Profile (within ceremony spec) | Differentiator | PassSign spec |
| 13 | Ceremony UX | Compose | Differentiator | PassSign spec conformance criteria |

Distribution: **8 Compose, 7 Profile, 4 Extend (two owned by others, two small and separable), 1 Invent — and the Invent item is protocol design.** No new cryptography anywhere. The decision-framework refutation condition for Q5 ("missing pieces require novel cryptographic primitives or unavailable platform APIs") does not hold: the baseline path uses zero unavailable APIs.

## Where PassSign emerges

The register shows the shape of the standard: **PassSign is (1) a signing ceremony specification** (intent, manifest, evidence record, re-binding, conformance UX — the Invent item), **(2) a set of small profiles** binding that ceremony to existing identity and signature standards (holder-binding PoP, status snapshots, evidence LTV), and **(3) an engagement agenda** for the two Extends that widen it over time (WebAuthn sign extension; SCAL2 recognition). Fragments of this gap are recognized across four communities — W3C, FIDO, ETSI, academia — and no one has claimed the composition (D2 III §4.3). First-mover framing is available.

## Go/no-go recommendation

**GO to Phase 5 (Protocol Proposal).** Conditions attached:

1. Baseline architecture is path (c1) — passkey-activated, certificate-bearing signing key with standard AdES output; assertion-evidence and native-sign-extension as additional conformance classes behind one ceremony abstraction.
2. The "open by default" principle requires protocol-level substitutability of the signing-service role (no single-operator dependency) — a named design requirement for Phase 5, since the baseline introduces a trusted operator.
3. Legal counsel review of tier claims before any public RFC; the synced-passkey/Art 26(c) question stays flagged, mitigated by the two-tier profile split.
4. Standards engagement (PR #2078 comment; W3C CG formation) should begin in parallel with Phase 5, not after it — the WebAuthn L4 use-case intake window is open now.

## Open questions handed to Phase 5

1. Evidence record format: JAdES-wrapped from day one, or a minimal JSON structure with a JAdES mapping annex?
2. Who operates signing services in the reference deployment model, and how does the protocol enforce substitutability (Chapter 12 vs condition 2)?
3. How does the re-binding ceremony (Chapter 10) authenticate "same person, new passkey" without re-running full identity proofing — and at which tier is that acceptable?
4. Multi-signer workflows: sequential/parallel signing order semantics in the manifest, or out of scope for v0.1?
5. Threat model priorities: malicious orchestrating RP is the hardest actor to constrain — what does the evidence record prove *against* the RP, not just through it?

---

*Regulatory findings herein inform protocol design and are not legal advice.*
