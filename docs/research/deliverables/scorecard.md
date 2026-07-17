# PassSign Consolidated Scorecard — Phases 1–3 Gate Review

Version 0.1 · 2026-07-17 · Covers Documents 1–3 (all six research streams)

## The hypothesis

> A universal, passkey-native signing protocol can be created by combining WebAuthn, Verifiable Credentials, existing digital signature standards, and established legal trust frameworks into a single open protocol that provides a significantly better user and developer experience than current electronic signing solutions.

## Verdict on the framing question

**PassSign is a novel composition of existing standards, not a new protocol — with one precise caveat.** Every load-bearing component classifies as Compose or Profile; nothing requires new cryptography. The caveat: WebAuthn *as deployed* cannot produce document signatures, so the composition must route signing through either (i) a passkey-authorized signing key held elsewhere (compose — all standards exist) or (ii) the in-flight WebAuthn `sign` extension (extend — being standardized by others, ~2028+). The naive reading of "passkey-native" (the passkey key signs the PDF) is refuted; the architectural reading (the passkey ceremony is the sole authorization and evidence root of every signature) is strongly supported.

## Q1–Q5 scorecard

| # | Question | Verdict | Key evidence | Doc |
|---|---|---|---|---|
| 1 | Technically possible? | **Supported (qualified)** | Assertions sign only `authenticatorData‖SHA-256(clientDataJSON)` — direct doc-signing excluded; ceremony-composed signing buildable today via CSC API v2.2 / ETSI TS 119 432 / EN 419 241-2; native path arriving via w3c/webauthn PR #2078 (verified first-hand) | Doc 2 |
| 2 | Legally viable? | **Supported** | SES everywhere; AdES defensible for device-bound + identity-binding profile (Art 26 text verified verbatim); QES via passkey-as-SCAL2-activation in the remote-QSCD model regulators already certify | Doc 3 |
| 3 | Interoperable? | **Supported on baseline path; profile work on evidence path** | Baseline output is byte-perfect PAdES/CAdES/JAdES validating in Adobe/EU DSS; direct-assertion evidence needs new JOSE alg/CMS OID + JAdES extension (specifiable; JAdES `etsiU` is the designed extension point) | Doc 2 |
| 4 | Sufficiently differentiated? | **Supported** | No incumbent connects passkeys to signing (Signicat sells both, unconnected); identity binding = email inbox at $0, real identity = $2.40/attempt; prior-art vacuum confirmed from two independent scans; narrowest in EU consumer segment (EUDI free QES) | Doc 1 |
| 5 | What's missing? | **All missing pieces are specifiable, none inventable** | See gap classification below | All |

## Gap classification (feeds Phase 4 Gap Analysis)

| Gap | Class | Notes |
|---|---|---|
| Signing ceremony semantics (intent, WYSIWYS policy, consent capture) | **Invent (protocol, not crypto)** — this is PassSign's core | SPC is the W3C precedent pattern |
| Evidence record binding passkey assertion + identity presentation + document hash | **Profile** | All formats exist; the record spec doesn't |
| Passkey-as-SAD binding for SCAL2 remote signing (challenge = DTBS/R hash) | **Profile / Extend** | Apparently unspecified anywhere — concrete near-term contribution to ETSI TS 119 432 / CSC |
| WebAuthn-assertion algorithm in JOSE/COSE/CMS registries + JAdES/PAdES carriage | **Extend** | Design straightforward (Doc 2 Part III §4.2); moat is process |
| X.509 issuance for passkey keys (assertion/attestation PoP for ACME/EST) | **Extend** | Separable small spec, broad utility |
| WebAuthn PoP profile for VC holder binding (`cnf` = passkey key) | **Profile** | Field-proven as hacks; OID4VCI proof-type extension point is the designed venue |
| Native raw signing from passkey authenticators | **Extend (owned by others)** | PR #2078 / WebAuthn L4; engage, don't build on before ~2029 |
| Identity binding, selective disclosure, revocation, timestamping, LTV | **Compose** | VC 2.0/SD-JWT (RFC 9901)/OID4VP/mdoc; RFC 3161; ETSI LTV — used as-is |
| Trust registries | **Compose** | Mirror EU Trusted List / issuer-accreditation patterns; no new registry invented |

## Architecture and venue decisions recommended at this gate

1. **Gate 1 (technical path):** primitive-agnostic ceremony abstraction with three conformance classes; baseline = passkey-activated HSM/RSSP-held signing key (standard AdES output); assertion-evidence and native-sign-extension as second/third classes.
2. **Gate 2 (legal posture):** two assurance tiers as protocol profiles (synced = attribution/SES-grade; device-bound-attested = AdES-grade); QES via QTSP on-ramp, never rebuilt; authenticator class recorded as first-class attribute.
3. **Gate 3 (venue):** W3C Community Group as birthplace; immediate engagement on PR #2078; IETF individual drafts for CMS/JAdES building blocks; CSC Associate membership; ETSI profile deferred until traction. Year-one budget < €5,000 + 0.2–0.5 FTE.

## Verification status

- Verified first-hand this pass: eIDAS Art 26(a)–(d) exact wording (legislation.gov.uk retained text, identical to EU original); PR #2078 full design description, discussion quotes (Steele "gross" hacks, Bradley WSCD/attestation, Yubico prototype-only status, yubicolabs fork, WebKit remote-cryptokeys overlap) — all stream reports confirmed accurate.
- Verified by streams against primary sources: WebAuthn L3 signed-payload structure; RFC 5652/7515 signing inputs; CSC API v2.2 and ETSI TS 119 432 V1.3.1 publication status; OID4VCI/VP Final status; RFC 9901; vendor pricing/product claims (fetched pages).
- Remaining **[unverified]** items (EUR-Lex returned empty pages to fetches): Art 5a(5)(g) free-QES exact wording (verified via mirror only); free-QES deadline (Dec 2026 vs Nov 2027); DocuSign EU Trusted List entries; country-by-country QES-mandatory lists; EUDI certified-wallet count mid-2026. None is load-bearing for the gate decision; all are flagged inline in the documents.

## Risks carried forward to Phase 4/5

1. Sign-extension slippage or narrowing (mitigated by primitive-agnostic abstraction).
2. No authoritative ruling on synced passkeys vs Art 26(c) — unquantified legal risk on the SES/AdES boundary (mitigated by two-tier profiles; monitor ETSI/ENISA).
3. EUDI free-QES commoditization in the EU consumer segment (mitigated by global/B2B/embedded positioning).
4. Baseline path requires a trusted service operator — the "open by default" principle must be preserved via protocol-level substitutability of the signing-service role (design requirement for Phase 5).

## Gate recommendation

**Proceed to Phase 4 (Gap Analysis).** All five questions score supported; no refutation condition from the decision framework holds; the composition verdict is clear enough that Phase 4's job is consolidation and enumeration rather than open investigation.

---

*Regulatory findings herein inform protocol design and are not legal advice; counsel review is assumed before RFC publication.*
