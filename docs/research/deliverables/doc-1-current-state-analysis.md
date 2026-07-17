# PassSign Document 1 — Current State Analysis

**Phase 3 deliverable (Streams 5–6: Existing Market · Standards Landscape)**
Version 0.1 · 2026-07-17 · PassSign Research Program

This document answers primary question **Q4 (Is it sufficiently differentiated?)** and settles the standardization-venue question. It consists of this synthesis followed by the two full stream research documents as Parts I–II.

---

## Executive synthesis

### The structural gap in the market

Three industries each hold one piece of the puzzle, and none composes them:

- **E-signature platforms** (DocuSign, Adobe, Dropbox Sign, Yousign, Signicat) bind signer identity by default to *access to an email inbox* — documented as insufficient in their own materials — and sell stronger identity as metered add-ons (e.g., $2.40 per verification attempt). None uses passkeys for anything, even login (Part I).
- **Wallet/identity providers** (Apple, Google, EUDI, Entra, Login.gov) have solved government-grade identity binding and cryptographic presentment at population scale — and expose no way to sign a document with it.
- **IAM providers** (Auth0, Okta) operate the world's passkey rails — and terminate every ceremony at session login. Okta even documents *blocking* synced passkeys for attestation reasons, a mainstream articulation of the assurance gap PassSign must handle.

The sharpest single data point: **Signicat sells a Passkeys product and an eIDAS signing API side by side, and they do not compose** — its "authentication-based signing" accepts eIDs but not the passkeys one menu over. The market has built the exact conceptual bridge PassSign proposes and left it unconnected.

### Evidence-model failure (the deeper differentiator)

Incumbent audit trails are platform-sealed logs: their probative value depends on trusting the vendor's process and continued existence. A signer holds no key; the platform's log *is* the evidence. A passkey-native protocol inverts this — signer-held keys producing verifier-checkable evidence independent of the platform's survival — while inheriting FIDO's phishing resistance and eliminating the emailed capability-URL attack surface incumbents themselves warn about.

### Collision scan: what already exists (and what doesn't)

- The **WebAuthn `sign` extension** (w3c/webauthn PR #2078, L4 milestone, ~2028+) standardizes the *primitive* — raw signatures with passkey-associated keys. Nobody found anywhere is standardizing the *composition*: ceremony semantics, AdES/PDF container mapping, identity binding, multi-party workflow. That layer is PassSign's defensible novelty.
- **EUDI Wallet free QES** (mandated, rolling out 2026–2027) is the strongest substitute threat — but only in the EU, only for consumer/non-professional use, and on a wallet-app + server-held-key architecture. The openings: rest-of-world, browser-native/no-app UX, and B2B/SaaS embedded signing.
- IETF Datatracker and GitHub scans return **zero** passkey-document-signing standards efforts — the prior-art vacuum is confirmed from a second independent direction.

### Venue recommendation (Gate 3 decision)

**Incubate as a W3C Community Group** ("Passkey Document Signing CG"): zero cost, immediate start, royalty-free IPR trajectory browser vendors require, and direct positioning as the use-case supplier for the WebAuthn L4 sign extension exactly while the 2026–2028 charter solicits such input (scope item 11). Four concurrent liaison threads: (1) engage PR #2078 design now — the highest-leverage action available in 2026; (2) 1–2 IETF individual drafts for transportable pieces (CMS/JAdES attributes carrying WebAuthn context); (3) CSC Associate membership (~€2,250–3,000) to profile passkey-authorized signing against CSC API v2.2; (4) ETSI TS profile later — ETSI ratifies traction, it doesn't birth it. Explicit anti-recommendation: FIDO Alliance and ETSI as birthplaces (pay-to-play, adjacent layers). Year-one standards budget: under €5,000 plus 0.2–0.5 FTE of sustained editorial participation.

Precedents validate the pattern: OIDC-on-OAuth, the FIDO2 W3C/CTAP split, OID4VC profiling three bodies' formats, and Secure Payment Confirmation — a chartered W3C ceremony layered on WebAuthn for a vertical, structurally the closest analogue to PassSign.

### Adoption map

Most capable early adopter: Signicat (all pieces in-house). Highest incentive: Auth0/Okta (new product line, zero cannibalization). Most natural mass channel: Dropbox Sign-tier API vendors (only path to AdES-class assurance without building an identity business) and government portals already running passkey MFA (Login.gov). Least likely: DocuSign/Adobe (identity add-ons are revenue; envelopes are the moat) — until it becomes table stakes.

### Scorecard

| Question | Verdict | Basis |
|---|---|---|
| **Q4 — Sufficiently differentiated?** | **Supported.** No incumbent or in-flight standard delivers passkey-native signing; the identified gaps (identity-binding economics, evidence portability, passkey/signing split) map to capabilities PassSign uniquely composes. Differentiation is narrowest in the EU consumer segment (EUDI free QES) — positioning must be complement, not competitor. | Parts I–II |

---

*The two parts follow, each with full profiles, matrices, venue analysis, and sources.*

---

# Stream 5: Existing Market

**PassSign research program — incumbent analysis: e-signature platforms, identity/wallet providers, IAM vendors**
Date: 2026-07-14. Method note: general web search engines (DuckDuckGo, Bing, Brave, Mojeek) were unreachable from this research environment; all findings below are drawn from direct fetches of vendor documentation, pricing pages, and official EU/US government pages (URLs in Sources). Claims that could not be verified against a fetched primary source are explicitly flagged **[unverified — from prior knowledge, needs confirmation]**. This document is market research, not legal or financial advice.

---

## Headline findings

1. **No incumbent connects passkeys to signing.** Signicat — the vendor closest to having all the pieces — sells Passkeys strictly as a *login* product (part of ReuseID) and delivers signatures through a separate Electronic Signing API whose identity methods are national eIDs, OTP, and video identification. Passkeys do not appear anywhere in its signing identity-method catalogue [1][2][3].
2. **The default identity binding across the mainstream market is still "access to an email inbox."** DocuSign says so explicitly in its own FAQ ("the standard way of identifying a signer is to send a link to the signer's email address") [6]; Adobe documents the same default and candidly describes its email OTP as "single-factor" because the code goes to the same inbox as the signing link [7].
3. **Stronger identity is sold as a metered, premium add-on** — e.g., DocuSign ID verification from **$2.40 per verification attempt** [5]; Adobe's phone/KBA/Government-ID checks are "premium authentication methods… a metered resource that must be purchased prior to use" [7].
4. **QES exists but is gated and high-friction**: enterprise-only plan tiers (DocuSign: eIDAS "AeS, SeS, QeS" only on contact-sales Enhanced plans [5]), certified-agent review steps (DocuSign ID Verification for EU Qualified [6]), and per-signature add-on pricing (Yousign, Signicat) [1][10].
5. **The wallet layer (Apple, Google, EUDI, Entra) proves identity but cannot sign documents; the IAM layer (Auth0, Okta) holds the passkey infrastructure but has no signing product.** The document-signing layer has neither. That three-way gap is the white space.

---

## E-Signature Platforms

### DocuSign

- **Problem solved:** market-leading envelope workflow — send, route, sign, store agreements; now repositioned as "Intelligent Agreement Management" (IAM) with AI document analysis [4].
- **Signature tier:** SES by default. eIDAS row in DocuSign's own plan matrix: Personal/Standard/Business Pro = "SeS" only; "AeS, SeS, QeS" appears only under contact-sales Enhanced plans [4]. QES is delivered via **ID Verification for EU Qualified**, which per DocuSign's FAQ requires an asynchronous review by a certified agent ("face-to-face or equivalent" under eIDAS), typically "a couple of minutes" [6]. DocuSign entities are on EU trusted lists as QTSPs **[unverified — from prior knowledge; check EU Trusted List browser]**.
- **Identity verification:** default is the emailed signing link; the Identify portfolio adds email access codes, SMS/phone OTP, knowledge-based authentication (US only), government-ID document + biometric liveness, CLEAR, eIDs, and a reusable "Identity Wallet" for repeat signers [6]. Risk-Based Verification (dynamic step-up) was in beta with GA "later in 2025" at time of page publication [6].
- **Developer API:** mature eSignature REST API, SDKs, Postman collections, free non-expiring demo sandbox; production requires a **go-live review process** and a paid API plan [5]. Developer plans: Starter $50/mo (40 envelopes/mo), Intermediate $300/mo (100/mo), Advanced $480/mo; embedded signing and IDV are only on contact-sales tiers [5]. Effective floor cost ≈ $1.25–3.00 per envelope at plan volumes, before identity add-ons.
- **Pricing/lock-in:** subscription + envelope allowance model. Web plans: Personal $11/mo (5 envelopes/mo), Standard $30/user/mo, Business Pro $45/user/mo — each capped at ~100 envelopes/user/year with pay-as-you-go overage billing; annual commitment enforced even on "billed monthly" plans [4]. Add-ons: SMS delivery from $0.36/delivery; IDV from $2.40/attempt [4]. Templates, PowerForms, Connect webhooks, and stored agreements create switching costs.
- **Documented failure modes:** (a) email-link identity is acknowledged by DocuSign itself as insufficient for "high-value or highly-sensitive agreements" [6]; (b) advanced identity and QES locked behind sales negotiation; (c) envelope-metered pricing penalizes high-volume/embedded use; (d) DocuSign maintains a dedicated "Safety Center" because DocuSign-branded phishing emails are a recognized attack vector [6 — site navigation]; (e) developer complaints about OAuth consent flows and go-live friction are common on Stack Overflow's `docusignapi` tag (DocuSign's own support FAQ routes developers there) [5] **[specific complaint volume unverified — search unavailable]**.
- **Passkeys:** no passkey or WebAuthn capability appears anywhere in the fetched pricing, product, or Identify documentation — neither for signer authentication nor for account login [4][5][6].

### Adobe Acrobat Sign

- **Problem solved:** enterprise e-signature embedded in the Acrobat/Creative ecosystem and large-enterprise document workflows.
- **Signature tier:** SES natively; AdES/QES via **cloud-based digital signatures**, where the signer authenticates to an external third-party signature/identity provider and Adobe "acts only as a platform for the digital signature to be requested and provided, with no additional costs added by Adobe" — the customer contracts with the provider directly [7]. (This is the Cloud Signature Consortium architecture; Adobe co-founded CSC **[unverified — from prior knowledge]**.) Aadhaar signing is purchasable through Adobe as an add-on for VIP-licensed accounts [7].
- **Identity verification ladder (documented in full):** none (email link) → Acrobat Sign account login ("not a second-factor method") → one-time password via email (explicitly described as single-factor because it reaches the same inbox) → signing password → phone OTP → KBA (US only) → Government ID + selfie → federated Digital Identity Gateway → cloud digital signature providers [7].
- **Premium metering:** "*Phone*, *KBA*, *Government ID*, and *Cloud-based digital signatures* are 'premium' authentication methods… a metered resource that must be purchased prior to use"; new enterprise accounts get 50 free phone/KBA transactions [7]. Admin guidance openly recommends configuring cheaper methods for internal recipients to avoid premium costs [7].
- **Developer API:** Acrobat Sign REST API with developer edition; integration is widely considered heavier than Dropbox Sign's **[comparative DX claim unverified — review sites unreachable]**. Audit reports record which authentication method was used; if "None," the report only shows that the document was signed [7].
- **Pricing/lock-in:** seat licensing bundled with Acrobat; enterprise VIP licensing; premium-auth transaction packs create per-signer marginal cost.
- **Failure modes:** same email-default weakness as DocuSign (documented in Adobe's own text); QES requires a second commercial relationship with an external IdP/TSP; KBA is US-only; per-transaction identity metering discourages routine strong authentication.
- **Passkeys:** absent from the entire authentication-methods documentation [7].

### Dropbox Sign (formerly HelloSign)

- **Problem solved:** developer-friendly embedded e-signature for SaaS products; positions on speed of integration ("integrate eSignatures in your app in just a few hours") [8].
- **Signature tier:** SES only. The FAQ actively distinguishes "eSignature API" from "digital signature API" and offers no AdES/QES, no eIDAS tiering, no identity-verification products [8].
- **Identity verification:** email delivery plus optional access codes/SMS **[SMS option unverified on fetched page]**; audit trail is "affixed to each signature request… tracked and time-stamped" [8].
- **Developer API:** free full-featured test mode without sales contact; API pricing: Essentials $75/mo (50 requests/mo), Standard $250/mo (100/mo, branding, embedded signing), Premium custom; >500 requests/mo requires sales [8]. G2-ranked "easiest to implement" per vendor claim [8].
- **Pricing/lock-in:** lowest entry cost of the majors; ~$1.50–2.50 per signature request at plan volumes. Lock-in mild (templates, embedded flows).
- **Failure modes:** ceiling problem — customers needing identity assurance or EU qualification must leave the platform entirely; pure email-identity binding; US-centric legal framing.
- **Passkeys:** none documented [8].

### Yousign (rebranding to "Youtrust")

- **Problem solved:** European SMB-focused e-signature with native eIDAS tiering and an expanding KYC/verification product line ("Yousign Verify": identity, company, bank-account, document-fraud, watchlist) [9][10]. The company is in the middle of a rebrand to **Youtrust**, signalling a move from signature vendor to trust-services platform [10].
- **Signature tier:** SES / AES / QES plus simple/advanced/qualified **eSeals** — all as first-class products [9][10]. Yousign operates as its own **Certification Authority** and publishes its eIDAS conformity certificates (LSTI-audited), including a qualified-signature CA certificate dated June 2025, plus qualified seal and qualified timestamp certificates [11]. It is a French QTSP on the EU Trusted List **[trusted-list entry itself unverified — list browser is JS-only; the published LSTI certificates corroborate]**.
- **Identity verification:** tier-mapped: SES = optional OTP; AES = mandatory OTP + ID verification; QES = ID verification incl. "face-to-face" (remote equivalent) verification [9]. QES flow therefore carries the standard eIDAS identity-proofing friction (document capture + verification session) before signing.
- **Developer API:** API-first pricing: Plus €104/mo (500 simple e-signatures/yr), Pro €129/mo (adds custom branding/notifications, API logs, 99.9% SLA), Scale custom; overage **€2.00 per signature**; AES/QES/eSeal/Verify are paid add-on packs; free 40-day full-featured sandbox; iFrame embedding, webhooks, smart anchors [10].
- **Audit trail:** time-stamped PDF evidence file archived **10 years** [10].
- **Pricing/lock-in:** annual billing only; signature-counted (per signer, not per envelope: one doc with 3 signers = 3 signatures) [10].
- **Failure modes:** per-signature economics at scale; QES priced as an opaque add-on ("contact us"); EU-only identity methods; SES still OTP/email-based.
- **Passkeys:** none documented anywhere on product, levels, pricing, or certification pages [9][10][11].

### Signicat (incl. Dokobit)

- **Problem solved:** pan-European digital identity hub — identity proofing (eID & Wallet Hub, VideoID, ReadID NFC), authentication (incl. Passkeys), electronic signing, qualified timestamps, digital evidence management — one vendor, ~35+ eID methods (Norwegian/Swedish BankID, MitID, itsme, DigiD, FTN…) [2][3].
- **The passkey lead, verified:** Signicat's Passkeys product is **authentication-only**: "a modern, passwordless authentication solution for browser-based sign-ins," delivered as part of **ReuseID** (its verified-identity store), with REST APIs, no-code flows, and RiskFlow orchestration [1]. Marketing use cases are logins (fintech, e-commerce, gaming, government) — signing is never mentioned [1]. In the developer docs, Passkeys sits under *Authentication*; *Electronic Signing* is a separate product family [2].
- **Signature tier:** all three eIDAS levels via the Sign API [3]:
  - **SES** — SMS/email OTP or scribble; no certificate; protected by Signicat's seal + timestamp.
  - **AES** — "authentication-based signing" with substantial/high eIDs, or identity built from scratch via SignatureID + VideoID; one-off or permanent signer certificates.
  - **QES** — AES + QSCD + qualified certificate via a QTSP; one-off certificates issued per signing session from high-level eIDs or VideoID High; third-party QES services (EstEID, itsme, BeID) [3].
- **Analytical point:** Signicat's own architecture shows the conceptual bridge PassSign proposes — "authentication-based signing," where a strong authentication event is converted into a signature with the platform's seal — but the accepted authenticators for signing are *eIDs*, not the passkeys it sells one menu over. A verified ReuseID identity can hold a passkey, yet that passkey cannot today produce even an AES. This is the single sharpest gap found in the market.
- **Developer API:** self-service free developer account, Docusaurus docs with `llms.txt`/markdown endpoints (notably good DX), dashboard, sandbox [2].
- **Pricing/lock-in:** per-transaction identity/signing pricing via sales; pricing page exists but headline rates not published for signing **[fetched pricing page not itemized]**.
- **Failure modes:** breadth = complexity (four product families, orchestration layer); eID coverage is Europe-only; QES still per-session certificate issuance with eID/video friction.

---

## Identity & Wallet Providers

### Apple Wallet ID / Digital ID

- **What it is:** ISO/IEC 18013-5 mDL and national IDs in Apple Wallet, presented in person, in apps via the **Verify with Wallet** PassKit API, and (stated intent) in browsers via the W3C Digital Credentials mdoc request API [12]. Coverage as of the fetched page: ~16 US states/territories, Japan My Number card, and **Digital ID derived from a U.S. passport (iOS 26.1+)** — the November 2025 expansion [12].
- **Identity binding:** issuing authority verifies the document + user during provisioning; presentment requires the same Face ID/Touch ID used at enrollment; responses are HPKE-encrypted, device-signed and issuer-signed, verified server-side against state IACA root certificates [12].
- **Signing capability:** none. "Signature" appears only as a *data element* (the image of your signature on the license). The API is entitlement-gated to 12 approved categories (financial services, car rental, alcohol delivery, etc.) — a closed, permissioned verifier ecosystem [12].
- **Relevance to PassSign:** Apple has solved government-grade identity binding + biometric ceremony + cryptographic presentment on 1B+ devices, but exposes no way to bind that identity to a document signature. Passkey (iCloud Keychain) and Wallet ID are separate silos.

### Google Wallet ID passes

- ISO-standard mdocs ("built on an open ISO standard… can be used with any other Wallet that implements the same standard"); issuer-signed, verified via cryptographic signature; in-person via NFC/QR; online/in-app acceptance rolling out [13]. Use cases: age verification, identity verification, driving privileges [13]. No signing capability; same silo structure as Apple.

### Microsoft Entra Verified ID

- W3C Verifiable Credentials + DIDs (`did:web` trust system), Microsoft Authenticator as wallet, issuance/verification REST APIs, OpenID4VC-family presentation protocols (SIOP, Presentation Exchange) [14].
- Solves *attested claims* (employment, identity attributes), not document signatures. The user "signs a verifiable presentation with their DID" — cryptographic signing exists in the stack, but is applied to credential presentations only; no document-signing product, no AdES/QES claims [14].
- Strategic note: an enterprise VC wallet that already signs presentations is technically adjacent to signing document hashes; Microsoft has not productized this.

### Login.gov (US GSA)

- Federated authentication + identity proofing for US federal agencies. MFA menu explicitly includes **"face or touch unlock" (passkeys)** and security keys, flagged as "more secure against phishing and theft," alongside PIV/CAC, TOTP, SMS, backup codes [15].
- Identity proofing: government ID photos + selfie, address verification by mail, or in-person at participating post offices (IAL2-oriented) [15].
- No signing service. Notable UX failure mode documented by Login.gov itself: lose all authenticators → **account cannot be recovered; user must delete it and start over** [15].
- Relevance: a government IdP already binding passkeys to proofed identities at population scale — an obvious future consumer of a passkey-signing protocol (e.g., signing federal forms).

### European Digital Identity Wallet (EUDI)

- Regulation (EU) 2024/1183 in force since May 2024; Member States must provide wallets to citizens — the Commission page states full rollout **"by the end of 2026"** [16]. Toolbox = Architecture & Reference Framework (ARF, continuously updated) + open-source reference implementation + technical standards; four Large-Scale Pilots covered mDL, eHealth, payments, education [16].
- Status signals as of the fetched page (updated July 2025, news through April 2026): ENISA tasked with wallet certification support (2024); EU age-verification blueprint on wallet specs (July 2025); **European Business Wallets** proposal (Nov 2025) [16]. **No Member State wallet was confirmed as certified in the sources fetched; certification status as of mid-2026 needs verification once search access is restored.**
- Critically for PassSign: the regulation obliges wallets to support creation of **qualified electronic signatures free of charge for natural persons** for non-professional use **[from Regulation 2024/1183 text — from prior knowledge of the regulation; the fetched Commission page does not restate it]**. The EUDI wallet is thus the only actor in this survey with a mandate to make QES a native, no-cost citizen capability — but on a wallet-app + national-eID architecture, not passkeys/WebAuthn.

---

## IAM Providers

### Auth0 (Okta company)

- Passkeys for database connections across Universal Login, embedded login, and native iOS/Android; synced credentials, autofill, cross-device QR flows; RP ID configurable to parent domain to share passkeys across web + mobile; limit 20 passkeys/user; post-login Actions can skip MFA after passkey auth [17].
- Pure authentication: the WebAuthn ceremony outputs a login, never a document signature. No signing product, no notion of signing-time evidence beyond tenant logs [17].
- **Could they add signing?** Technically well positioned: they run the RP infrastructure, credential storage, and ceremony UX. Missing: document/hash payload semantics, evidence formats (AdES containers), trust-service status.

### Okta (Workforce)

- FIDO2 (WebAuthn) authenticator with user-verification policy, authenticator groups from FIDO Metadata Service (FIPS status, hardware protection), admin-enrolled security keys for onboarding [18].
- Most revealing feature: **"Block the use of passkeys"** — Okta lets orgs block synced passkeys precisely because they "don't have some enterprise security features, such as device-bound keys and attestations" [18]. This is a documented, mainstream articulation of the assurance gap PassSign must address: synced passkeys trade attestation/device-binding for recoverability, which matters when a credential must anchor a legally significant signature.
- No signing product; same adjacency argument as Auth0.

---

## Capability Matrix

| Product | Identity binding (default → max) | Max signature tier | Passkey use | Open protocol? | API ergonomics | Pricing model |
|---|---|---|---|---|---|---|
| **DocuSign** | Email link → GovID+biometric / eID / certified-agent QES review | QES (Enhanced plans + EU Qualified IDV) [4][6] | None documented | No — proprietary envelopes/Connect | Mature REST, SDKs, free sandbox, go-live gate | Sub + envelope caps; IDV $2.40/attempt; SMS $0.36 [4][5] |
| **Adobe Acrobat Sign** | Email link → GovID+selfie / cloud digital ID via external TSP | QES (via third-party cloud signature providers) [7] | None documented | Partially — CSC for cloud signatures **[CSC role unverified]** | Heavier REST; premium auth metered | Seats + prepaid premium-auth transactions [7] |
| **Dropbox Sign** | Email link (+ access codes) | SES only [8] | None documented | No | Best-in-class simplicity; free test mode | $75–$250/mo, 50–100 req/mo, sales above 500 [8] |
| **Yousign / Youtrust** | Email/OTP → ID verification → face-to-face-equivalent (QES) | QES (own QTSP/CA, LSTI-certified) [9][11] | None documented | No | Clean REST, iFrame, 40-day sandbox | €104–129/mo + add-on packs; €2/sig overage [10] |
| **Signicat** | Email/SMS OTP → 35+ eIDs → VideoID High | QES (one-off certs, QTSP network, QTSA) [3] | **Login only** (ReuseID Passkeys) [1] | Standards-based eID rails; proprietary APIs | Strong docs (llms.txt), free dev account | Per-transaction, sales-led |
| **Apple Wallet ID** | Government issuance + Face/Touch ID presentment | n/a (no signing) | Separate silo (iCloud passkeys ≠ Wallet ID) | ISO 18013-5 / W3C DC API (intent) [12] | Entitlement-gated PassKit; 12 approved categories | Free API; gatekeeping, not pricing |
| **Google Wallet ID** | Government issuance + device auth | n/a (no signing) | Separate silo | ISO mdoc [13] | Standard Google dev flow | Free |
| **Entra Verified ID** | Issuer-attested VCs; wallet = Authenticator | n/a (signs VPs, not documents) [14] | No (separate Entra passkey features) | W3C VC/DID, OpenID4VC family | REST issuance/verification APIs | Entra licensing |
| **Login.gov** | IAL2 proofing (ID + selfie / in-person) | n/a (no signing) | **Yes — passkeys as MFA** [15] | SAML/OIDC federation | Agency-only | Government |
| **EUDI Wallet** | National eID / PID, notified schemes | **QES mandated, free for citizens** (per Reg. 2024/1183) **[unverified restatement]** | Wallet apps, not WebAuthn passkeys | Yes — ARF, open reference implementation [16] | Open-source libs, GitHub | Public infrastructure; rollout due end-2026 |
| **Auth0** | Account identity only | n/a | **Yes — login only** [17] | WebAuthn (open) | Excellent | Per-MAU |
| **Okta** | Workforce identity; attestation-aware | n/a | **Yes — login only; synced passkeys blockable** [18] | WebAuthn/FIDO2 (open) | Good | Per-user |

---

## Failure-Mode Inventory

**Identity binding**
1. *Email ≈ identity* is the industry default; both DocuSign and Adobe document it as the baseline and document its weakness (forwarded links, shared inboxes, compromised email) [6][7]. The audit trail then records "the person who controlled this inbox at this time," not a person.
2. Strong identity is *per-transaction priced* ($2.40 DocuSign IDV; Adobe prepaid premium auth), so senders rationally under-verify [4][7].
3. Verification evidence is *not portable*: a signer who did GovID+selfie for DocuSign must redo it for Adobe, Yousign, and every bank (partially mitigated by vendor-proprietary reusable identities: DocuSign Identity Wallet, Signicat ReuseID [6][1] — which deepen lock-in rather than solve portability).

**Audit trails as evidence**
4. Audit trails are platform-generated logs (certificate of completion, PDF evidence file) sealed by the *platform's* key — probative value rests on trusting the vendor's process, its retention (Yousign: 10 years [10]), and its continued existence; the artifact is not independently verifiable cryptography by the signer's own key (except in eID/QES flows).
5. Adobe's audit report shows only "document was signed" when authentication is set to None [7] — the floor of evidentiary quality is very low and configuration-dependent.

**Developer experience**
6. DocuSign: sandbox is free but production is gated by a go-live review plus $50–480/mo plans with envelope meters; embedded signing only at the top tier [5]. Adobe: premium-auth procurement through sales. Dropbox Sign is easy but capability-capped at SES [8]. Net effect: **assurance level and DX are inversely correlated** across the market.
7. Per-envelope/per-signature metering (DocuSign PAYG overage [4]; Yousign €2/signature overage [10]) makes signing a COGS line item that scales linearly — hostile to high-frequency, low-value signing (approvals, consents, gig-economy onboarding).

**End-user UX**
8. Occasional signers face compounding friction at higher tiers: account creation (Acrobat Sign authentication requires an Adobe account [7]), OTP juggling, document + selfie capture, and for QES a certified-agent review wait [6] and per-session one-off certificates [3].
9. Recovery cliffs: Login.gov documents that losing authenticators means deleting the account [15]; Okta documents blocking synced passkeys for attestation reasons [18] — recoverability vs. assurance is unresolved even in pure authentication, and worse for signature keys.
10. Phishing: signature requests arrive as emails from platform domains; DocuSign runs a Safety Center because its brand is a top phishing lure [6]. An email-launched signing ceremony inherits email's threat model.

**Passkey/signing split (the structural failure)**
11. Every passkey deployment found (Signicat, Auth0, Okta, Login.gov) terminates at session login. Every signature product found binds identity via email/OTP/eID/video instead. Even within a single vendor (Signicat) the two products do not compose [1][2][3].

---

## Opportunity Map

**What a passkey-native open signing protocol could do that no incumbent does:**

1. **Signer-held keys, verifier-checkable evidence.** Today the signer holds no key (except in eID/QES flows); the platform's log *is* the signature's evidence. A protocol where the WebAuthn credential's private key signs a commitment to the document hash yields evidence any relying party can verify against the credential's public key — independent of the platform's survival or honesty. (Design caveat for Streams 1/3: WebAuthn's challenge is opaque to the user; a WYSIWYS/"what-you-sign-is-what-you-see" layer and payload-binding convention are required, and synced-passkey attestation limits — the exact issue Okta documents [18] — must be addressed for higher assurance tiers.)
2. **Decouple identity proofing from signing, once.** Bind a wallet-verified identity (Apple/Google mdoc presentment [12][13], EUDI PID [16], Entra VC [14]) to a passkey at enrollment; thereafter every signature inherits that binding at zero marginal identity cost — directly attacking the $2.40-per-attempt economics [4].
3. **Kill the account-creation and email-link dependency.** A passkey ceremony needs no vendor account and no emailed capability URL; phishing resistance is inherited from FIDO's origin binding — the strongest documented property incumbents lack (their own docs recommend OTP precisely because email links leak [7]).
4. **A ladder into eIDAS.** SES-equivalent immediately (unique link to signer + sole-control key arguably reaches AdES characteristics — legal analysis belongs to Stream 4); QES via the EUDI wallet's mandated free signing function or via a QTSP treating the passkey ceremony as the signature-activation authentication.

**Who could adopt it:**
- **Signicat** — has passkeys, identity proofing, a signing API, and QTSA/QTSP relationships; adopting would mean connecting two existing menus [1][2][3]. Highest capability, moderate incentive (per-transaction eID revenue).
- **Auth0/Okta** — own the passkey rails and developer mindshare; a signing extension is a new product line with no cannibalization. Highest incentive among IAM.
- **Dropbox Sign–tier API vendors** — an open protocol is their only realistic path to AdES-class assurance without building an identity business.
- **Login.gov / government service portals** — passkey MFA is already deployed [15]; signing federal/citizen forms is the natural next step.
- **DocuSign/Adobe** — least likely early adopters (identity add-ons are revenue; envelopes are the moat), but both have precedent for standards adoption when it becomes table stakes (Adobe with cloud-signature federation [7]).
- **EUDI ecosystem** — if the protocol profiles cleanly against ARF/QES flows, wallet providers get a browser-native signing UX; conversely EUDI is also the strongest substitute threat in the EU.

**Substitute/timing risk:** in the EU, the EUDI wallet (due end-2026 [16]) may deliver "free QES for citizens" through wallet apps before a passkey-native protocol matures — the opening is (a) the rest of the world, (b) the browser-native/no-app UX, and (c) B2B/SaaS embedded signing where wallet ceremonies are too heavy.

---

## Sources

1. Signicat — Passkeys product page: https://www.signicat.com/products/authentication/passkeys
2. Signicat — Developer documentation portal (product taxonomy: Passkeys under Authentication; Electronic Signing separate): https://developer.signicat.com/
3. Signicat — Electronic Signature API (SES/AES/QES mechanisms, identity methods, certificates): https://www.signicat.com/products/electronic-signing/electronic-signature-api
4. DocuSign — eSignature plans & pricing (envelope allowances, eIDAS tier row, add-on pricing): https://ecom.docusign.com/plans-and-pricing/esignature
5. DocuSign — Developer API plans & pricing (sandbox, go-live, envelope meters): https://ecom.docusign.com/plans-and-pricing/developer
6. DocuSign — Identify / ID verification product page & FAQ (email-link default, IDV portfolio, EU Qualified agent review): https://www.docusign.com/products/identify
7. Adobe — Acrobat Sign Identity Authentication Methods (full auth ladder, premium metering, cloud digital signatures): https://helpx.adobe.com/sign/using/signer-identity-authentication-methods.html
8. Dropbox Sign — eSignature API product & pricing page: https://www.dropboxsign.com/products/dropbox-sign-api
9. Yousign — eSignature levels (SES/AES/QES identity requirements): https://yousign.com/electronic-signature-levels
10. Yousign — API pricing (plans, overage, add-ons, audit trail retention, Youtrust rebrand): https://yousign.com/pricing-api
11. Yousign — Digital signature certificates (CA status, LSTI eIDAS certificates incl. qualified signature CA 2025): https://yousign.com/digital-signature-certificate
12. Apple — Get started with the Verify with Wallet API (coverage incl. U.S.-passport Digital ID/iOS 26.1, ISO 18013-5, entitlement categories): https://developer.apple.com/wallet/get-started-with-verify-with-wallet/
13. Google — Google Wallet Identity developer documentation: https://developers.google.com/wallet/identity
14. Microsoft — Introduction to Microsoft Entra Verified ID: https://learn.microsoft.com/en-us/entra/verified-id/decentralized-identifier-overview
15. Login.gov — Authentication methods help page: https://www.login.gov/help/create-account/authentication-methods/
16. European Commission — EU Digital Identity Wallet Toolbox process (ARF, rollout by end-2026, ENISA certification, Business Wallets news): https://digital-strategy.ec.europa.eu/en/policies/eudi-wallet-toolbox
17. Auth0 — Passkey Authentication for Database Connections: https://auth0.com/docs/authenticate/database-connections/passkeys
18. Okta — Configure the FIDO2 (WebAuthn) authenticator (incl. "Block the use of passkeys"): https://help.okta.com/oie/en-us/content/topics/identity-engine/authenticators/configure-webauthn.htm

*Research limitation:* web search engines were unreachable from this environment; independent review aggregators (G2, Stack Overflow complaint analysis), the EU Trusted List browser (JavaScript-only), and news coverage of EUDI wallet certification status could not be consulted. Items flagged **[unverified]** above should be confirmed in the consolidated verification pass (Task #10).


---

# Stream 6: Standards Landscape

**PassSign research program — Stream 6: who owns what, and where a PassSign spec should live**
Prepared: 2026-07-14. Method note: research was conducted against primary standards-body sources (datatracker.ietf.org, w3.org, fidoalliance.org, etsi.org, openid.net, cloudsignatureconsortium.org) plus GitHub's public API. General-purpose web search engines were not reachable from this environment (a DuckDuckGo HTML fetch returned empty), so all in-flight-work claims below are anchored to primary sources rather than secondary reporting. Claims marked "[reference]" in the Sources list are canonical spec locations cited from general knowledge rather than fetched during this session.

*Disclaimer: this document is a standards-landscape and governance analysis. It is not legal advice; venue, IPR, and eIDAS-related conclusions should be validated with counsel before commitments are made.*

---

## 1. Ownership Map

| Body | What it owns that PassSign touches | Membership / IPR model | Typical timeline |
|---|---|---|---|
| **FIDO Alliance** | CTAP 1/2 (authenticator protocol), UAF, U2F, authenticator/server certification programs, FIDO Metadata Service, Credential Exchange specs (CXP/CXF); new focus areas: **Digital Credentials** and **Agentic AI** [6][7] | Paid corporate membership via Membership Agreement (Board/Sponsor/Associate/Government tiers; fees set in the agreement); member-only working groups; patent policy aligned with W3C-style royalty-free commitments per the FIDO IPR Summary ("v4 W3C") [7] | Proposed Draft → Review Draft → Proposed Standard → Final; CTAP major revisions run 2–4 years |
| **W3C** | WebAuthn API (WebAuthn WG; Level 3 reached Candidate Recommendation 2026-01-13; Level 4 being chartered), Credential Management, Digital Credentials API (Federated Identity WG), VC Data Model 2.0, Secure Payment Confirmation (Web Payments WG) [3][4][5] | Corporate membership (fee scale by size/region [17]); **Patent Policy 2020: royalty-free licensing commitments**; free Community Group track with CLA/Final Specification Agreement providing RF commitments on Final Reports [12] | WG Recommendation: 2–5 years; CG report: weeks–months; WebAuthn L3 took ~4 years from FPWD to CR |
| **IETF** | CMS (RFC 5652), JOSE (RFC 7515–7519), COSE (RFC 9052/9053), OAuth 2.x, SD-JWT (OAuth WG), CFRG crypto algorithms (incl. ARKG, see §3) [1][11] | No membership; open participation under the Note Well; IPR is **disclosure-based (RAND-style), RF not guaranteed** but common in security area; anyone can post an Internet-Draft for free | Individual I-D: instant publication; WG adoption → RFC: typically 2–5 years |
| **ETSI (TC ESI)** | AdES signature formats (CAdES/XAdES/PAdES — EN 319 122/132/142), TSP policy standards (EN 319 401/411-1), remote signing: **TS 119 431 (server signing) and TS 119 432 (signature creation protocols; V1.3.1 published March 2026)**, trusted lists (TS 119 612); the standards backbone of eIDAS/eIDAS2 [8][9] | European SDO; membership fees scale with company size; participation also possible via national standards organizations; FRAND IPR policy; deliverables free to download | TS: 1–2 years when driven by an EC standardisation request; EN: longer (national vote) |
| **ISO/IEC** | PDF (ISO 32000-2, incl. PDF digital signature/DSS structures that PAdES profiles), ISO/IEC 18013-5 mDL and 18013-7 (online presentment), 23220 series (mobile eID architecture) [15][16] | National-body voting model; participation via national mirror committees; documents behind paywall; RAND IPR | 3–6 years per standard; TS faster |
| **OpenID Foundation** | OID4VCI, OID4VP, SIOPv2 and the **High Assurance Interoperability Profile (HAIP)** in the Digital Credentials Protocols (DCP) WG; a new **Digital Credentials Harmonized Presentation WG** appeared in 2025/2026; certification program [10][13] | Very low-cost: **US$50 individual, $1,000 for orgs ≤25 employees, up to $20,000 for >100**; per-WG IPR contribution agreement; specs royalty-free under OIDF IPR policy [13] | Implementer's Drafts in ~1–2 years; Final specs 2–4 years; fast iteration culture |
| **Cloud Signature Consortium** | **CSC API** — the de facto remote-signing API used by TSPs and adopted in the EUDI Wallet context; **v2.2 published 6 Nov 2025**; CSC Data Model v1.0.0 (Oct 2025); conformity checker & CSC CERTIFIED program [2][14] | Brussels-based VZW; Associate €3,000 / Premium €6,000 / Executive €10,000 per year, **25% discount for SMEs/non-profits/public sector**; specs publicly downloadable [14] | API minor revisions roughly annually (v2.0 2023 → v2.1 Jan 2025 → v2.2 Nov 2025) |

**What PassSign would touch in each.** A "signing ceremony/protocol composing WebAuthn + VC + AdES" crosses at least five ownership boundaries: the client API and ceremony (W3C WebAuthn / Digital Credentials API), the authenticator behavior (FIDO CTAP), the credential/attestation formats (W3C VC-DM, IETF SD-JWT, ISO mdoc, OIDF presentation protocols), the signature container (ETSI AdES over IETF CMS/JOSE/COSE and ISO PDF), and the server-side signing interface (CSC API, profiled by ETSI TS 119 432).

---

## 2. Composition Precedents

Cross-body composition is the norm, not the exception, in exactly this space. Three precedents are directly instructive:

**(a) OpenID Connect layered on OAuth 2.0 (OIDF + IETF).** IETF owns the authorization framework (RFC 6749); OIDF built the identity layer on top as its own spec suite with its own certification program. What made it work organizationally: (i) a clean layering contract — OIDC never modified OAuth wire semantics, only profiled and extended them; (ii) overlapping editors who were active in both bodies, providing informal liaison; (iii) OIDF's lightweight IPR (per-spec contribution agreement) and cheap membership let the same individuals participate in both venues. The lesson for PassSign: you do not need the base-layer body's permission to layer on its work, but you need people who sit in both rooms.

**(b) The FIDO2 split: W3C WebAuthn + FIDO CTAP.** FIDO Alliance developed the FIDO 2.0 web APIs, then submitted them to W3C (2015/2016), where the WebAuthn WG was chartered; FIDO retained the authenticator-facing protocol (CTAP). The split persists by design: the current WebAuthn WG charter lists the "FIDO2 Technical Working Group" as an external coordination body, and both charters cite each other's deliverables [3][4]. What made it work: browser vendors would only implement under the W3C royalty-free patent policy, while authenticator vendors and the certification business stayed inside FIDO's membership model. Each body kept the layer matching its IPR regime and its commercial constituency. The lesson: the *browser-visible* layer of PassSign must live where browser vendors' patent-policy requirements are met (W3C RF), while certification/conformance can live in a consortium.

**(c) OID4VC profiling W3C VCs and ISO mdocs (OIDF + W3C + ISO).** The DCP WG's charter is explicitly format-agnostic: OID4VP/OID4VCI carry "Digital Credentials of any format (W3C VCs, IETF SD-JWT VCs, ISO/IEC 18013-5, etc.)," and the profiling relationship runs in *both* directions — ISO/IEC 18013-7 and the 23220 series profile OID4VP for mdoc/mDL presentment, and the work is done "in liaison with the European Commission, DIF, ETSI, and ISO/IEC SC17 WG4 and WG10" [10]. The lesson: a protocol body can successfully compose data formats owned by three other bodies if it declares format-neutrality and institutionalizes liaisons in the charter.

**(d) A fourth, smaller precedent worth noting: Secure Payment Confirmation (SPC).** The W3C Web Payments WG built a *transaction confirmation ceremony on top of WebAuthn* — arguably the closest structural analogue to PassSign's signing ceremony — as a W3C spec layered on another W3C spec, coordinated with FIDO and EMVCo through the Web Payment Security Interest Group (a liaison venue named in the WebAuthn charter) [3][18]. The lesson: "WebAuthn + ceremony semantics for a vertical" already has a chartered W3C pattern.

---

## 3. In-Flight & Colliding Work (2024–2026)

This is the decision-critical section. There **is** already active standards work on generic signing with passkeys, and PassSign must be positioned relative to it rather than in ignorance of it.

### 3.1 W3C WebAuthn WG: the `sign` extension (direct overlap — highest relevance)

- **PR #2078 "Add 'sign' extension"** on w3c/webauthn, opened 28 May 2024 by Emil Lundberg (Yubico), still open and assigned to the **Level 4 First Public Working Draft milestone** (labeled `@Risk` for L3; last updated April 2026). It defines signing of **arbitrary data with a key associated with, but distinct from, a WebAuthn credential key pair**, explicitly motivated by (i) "attested, hardware-bound signing keys for applications such as digital identity wallets" and (ii) "using FIDO security keys (possibly unattended) for general-purpose digital signatures, with seamless interoperability with existing cryptographic protocols" [5]. It supersedes earlier approaches in issues #1895 and #1945 (WebCrypto/PRF-derived keys), which were rejected because such keys are not truly hardware-bound.
- **The draft 2026 WebAuthn WG charter makes this official scope.** New scope item 11: *"Signing additional data beyond the standard authentication data. For example, to support use cases in the agentic AI and verifiable digital credential ecosystems."* The recharter (2026–2028) targets WebAuthn Level 4, with L3 having reached CR on 2026-01-13 [4]. Note the charter still excludes "low-level access to credential private key material" — the sign extension threads this needle by using *derived* keys, not the credential key.
- **Supporting crypto at IETF:** `draft-bradleylundberg-cfrg-arkg-10` (Asynchronous Remote Key Generation), Lundberg (Yubico) & Bradley, active individual draft in CFRG, last updated 2026-02-27, developed at github.com/Yubico/arkg-rfc [11]. ARKG lets a relying party derive fresh public keys for a passkey without authenticator interaction — the privacy/key-separation mechanism underpinning the sign extension.

**Implication for PassSign:** the *primitive* PassSign needs — "passkey produces a raw signature over caller-chosen bytes" — is being standardized by others, on a W3C L4 timeline (charter runs to 2028). PassSign's defensible novel layer is everything *above* that primitive: the document-signing ceremony (what the user sees and consents to), binding the signature into AdES/PDF containers, attaching identity via VC/SD-JWT presentation, and the multi-party workflow (signer ↔ signing service ↔ verifier). Nobody found in this research is standardizing that composition.

### 3.2 FIDO Alliance beyond authentication

FIDO now maintains formal focus areas for **Digital Credentials** and **Agentic AI** alongside passkeys, plus the Credential Exchange (CXP/CXF) specifications for migrating credentials between providers [6][7]. Its DocAuth/Face Verification (IDV) certification programs extend the certification franchise beyond authenticators. No FIDO deliverable found addresses document signing, but the CTAP-side counterpart of the WebAuthn sign extension would land in the FIDO2 TWG (member-only), per the layering described in PR #2078 ("client-authenticator layer") [5].

### 3.3 ETSI TC ESI / EUDI Wallet signing

- **TS 119 432 (protocols for remote digital signature creation) V1.3.1 published March 2026** — a fresh revision, confirming active maintenance of the remote-signing protocol layer in the eIDAS2/EUDI timeframe [9].
- TC ESI's remit explicitly covers "TSP providing remote signature creation or validation functions" and support for the eIDAS Regulation [8].
- The **CSC published a 2026 report "EUID Wallet – How to implement Free QES"** [14], reflecting the eIDAS2 obligation that EUDI Wallets provide qualified electronic signatures free of charge for non-professional use — the single largest institutional driver of wallet-based signing today. The EUDI signing stack being assembled is: wallet authentication + CSC API to a remote QSCD + AdES formats — i.e., *server-held keys*, not passkey-local keys. PassSign's passkey-native model is architecturally different but will be judged against this stack in Europe.

### 3.4 Cloud Signature Consortium roadmap

CSC API v2.2 (Nov 2025) and the first release of a **CSC Data Model v1.0.0** (Oct 2025) show the consortium expanding from a bare API to a broader interoperability layer; its interoperability events and certification program continue [2][14]. CSC is the natural place where a PassSign-to-TSP bridge (passkey-authorized remote AdES sealing) would be profiled.

### 3.5 OpenID Foundation

The DCP WG continues OID4VCI/OID4VP/HAIP with formal security analyses (Univ. of Stuttgart, 2023 and July 2025) and liaisons to EC, ETSI, ISO SC17, DIF, OWF [10]. A **Digital Credentials Harmonized Presentation WG** now exists alongside DCP [13] — evidence that OIDF remains the fastest-moving venue for wallet/credential protocol composition. No OIDF work item on document signing was found.

### 3.6 Collision scan results (negative findings, which matter)

- **IETF Datatracker full-text search for "passkey"** returns only two drafts: `draft-sander-open-tabs-passkey-00` (passkey-rooted signers for Solana payment channels, June 2026) and the expired `draft-bucksch-sasl-passkey-00` (SASL mechanism) [1]. **No IETF draft attempts passkey-based document signing.**
- **GitHub repository search for "passkey signing"** surfaces only authentication libraries and one small tool (`ItalyPaleAle/revaulter`, "encrypt, decrypt, and sign with passkeys") — no standards-track or consortium-grade document-signing effort [19].
- Caveat: without a general web search engine this session, smaller blog-level proposals may have been missed; the primary-source scan (datatracker, W3C GitHub, FIDO, ETSI, CSC, OIDF) is authoritative for *standards-track* work.

---

## 4. Venue Analysis

### (a) IETF Internet-Draft → RFC
- **IP policy:** Note Well disclosure regime; no guaranteed royalty-free outcome (weaker than W3C for browser-adjacent tech).
- **Speed:** an I-D can be published today for $0; but RFC status realistically 3–5 years, and PassSign has no obvious home WG — a signing-ceremony spec would face DISPATCH/SECDISPATCH and likely a BoF. OAuth WG fits only the token/SD-JWT edges; LAMPS fits only CMS edges.
- **Audience:** protocol engineers; not browser vendors (they standardize web APIs at W3C, not IETF), not TSPs (they follow ETSI/CSC).
- **Charter fit:** poor for the ceremony; **good for narrow crypto/data-structure components** (e.g., a COSE/CMS signature attribute carrying WebAuthn assertion context — the natural companion to ARKG already in CFRG).
- **Precedent:** ARKG shows the passkey community already routes crypto primitives here [11].
- **Verdict:** wrong primary venue; right venue for 1–2 supporting building-block drafts.

### (b) W3C Community Group (→ Working Group)
- **IP policy:** best-in-class — CLA on contributions, Final Specification Agreement gives royalty-free commitments on the Final Report; explicitly "tuned for transition to standards-track" [12].
- **Speed:** fastest possible start: no fee, no charter, 5 supporters, public from day one; a CG Report can exist within months.
- **Audience:** exactly the actors who control the passkey signing primitive — the WebAuthn WG (whose new charter names the **Web Identity and Credentials CG** as its use-case intake channel [4]) and the FedID WG (Digital Credentials API). Browser-vendor engineers watch CGs; they do not watch ETSI or CSC.
- **Charter fit:** PassSign's ceremony layer is a web-platform composition; a CG has no charter constraints at all. The WebAuthn WG itself will not host PassSign (its charter is authentication + the signing *primitive*, and UX prompts are explicitly non-normative), which is precisely why an adjacent CG is the correct staging ground.
- **Precedent:** WebAuthn itself entered W3C as an external (FIDO) submission; Digital Credentials incubated in WICG before moving to the FedID WG; SPC shows a WebAuthn-layered ceremony spec succeeding at W3C [18].
- **Verdict:** **best primary venue for incubation.**

### (c) FIDO Alliance extension
- **IP policy:** RF-style per FIDO IPR summary, but development is member-only [7].
- **Speed/cost:** requires signing the Membership Agreement and paying an annual fee; agenda controlled by large platform members; a small company cannot drive a new work area.
- **Charter fit:** FIDO owns the authenticator layer and certification; its new Digital Credentials/Agentic AI focus areas show appetite, but a *document-signing ceremony* has no current FIDO home.
- **Verdict:** not the primary venue; join later (or via liaison) if PassSign needs CTAP-level behavior or a certification program.

### (d) ETSI profile (TS 119 series)
- **IP policy:** FRAND; fine for TSPs, unattractive for browser implementers.
- **Speed:** reasonable when EC-mandated; slow otherwise; participation requires ETSI membership or a national-body route — heavy for a small team.
- **Audience:** TSPs, auditors, EU regulators — the people who make PassSign *legally meaningful* under eIDAS, but who will not design a web ceremony.
- **Charter fit:** perfect for the endgame ("TS 119 4xx: AdES signature creation using FIDO2/WebAuthn-held keys" or a PassSign profile of TS 119 432), wrong for invention. ETSI standardizes what already has ecosystem traction.
- **Verdict:** target venue for a *later* profile, reached via liaison; not the birthplace.

### (e) OpenID Foundation WG spec
- **IP policy:** royalty-free under OIDF IPR policy; per-WG contribution agreement; individual membership $50 [13].
- **Speed:** proven fast (OID4VC family); certification machinery available.
- **Audience:** wallet vendors, identity providers, the EUDI ecosystem — strong overlap with PassSign's identity-binding layer (VC/SD-JWT presentation during signing).
- **Charter fit:** good **if** PassSign is framed as a protocol (issuer/holder/verifier flows, HTTP/OAuth-shaped); weaker for the browser-API-shaped ceremony. Chartering a new WG requires recruiting co-proposers, which the DCP community could supply.
- **Verdict:** strong secondary venue; the best fallback primary if PassSign turns out to be more protocol than platform API.

---

## 5. Recommended Venue & Engagement Strategy

**Primary recommendation: incubate PassSign as a W3C Community Group** (e.g., "Passkey Document Signing CG"), producing a CG Report that specifies the signing ceremony and the composition profile (WebAuthn `sign` extension + VC/SD-JWT identity binding + AdES/PDF container mapping). Rationale: zero cost, immediate start, royalty-free IPR trajectory acceptable to browser vendors, and — decisively — it positions PassSign as the *consumer and use-case supplier* for the WebAuthn L4 `sign` extension exactly when the WebAuthn WG's 2026–2028 charter is soliciting such use cases (scope item 11 and the WICA CG intake channel) [4][5].

**Liaison strategy (four concurrent threads):**
1. **W3C WebAuthn WG / PR #2078:** engage immediately on the sign-extension design (GitHub is open to non-members under the patent policy); PassSign's document-signing requirements (content-hash payloads, AdES-compatible signature algorithms, attestation needs) should shape the extension before L4 FPWD. This is the single highest-leverage action available in 2026.
2. **IETF:** publish 1–2 individual drafts for the transportable pieces — e.g., a CMS/COSE signed-attribute or JAdES header parameter that embeds WebAuthn authenticator data so an AdES validator can verify the passkey ceremony. Route via LAMPS (CMS) or COSE WG; cite ARKG [11].
3. **CSC:** Associate membership (€3,000, ~€2,250 with SME discount [14]) to profile "passkey-authorized signing" against CSC API v2.2 — this is the bridge to TSPs and to the EUDI "free QES" implementation wave.
4. **ETSI TC ESI (deferred):** once the CG Report and at least one implementation exist, pursue an ETSI TS profile (via a member sponsor or national body) so PassSign signatures can be assessed for AdES conformance and eventual eIDAS recognition. Do not attempt this first; ETSI ratifies traction.

**Explicit anti-recommendation:** do not attempt to make FIDO Alliance or ETSI the birthplace. Both are pay-to-play, member-driven venues where a small company cannot set agenda, and both standardize layers adjacent to — not identical with — PassSign's novel ceremony layer.

**Key risk to manage:** the WebAuthn sign extension is labeled `@Risk` and deferred to L4; if it slips or is narrowed to wallet-key derivation only, PassSign's client primitive weakens. Mitigation: keep the PassSign ceremony spec primitive-agnostic (define an abstraction that can bind to the sign extension, to largeBlob/PRF-style workarounds, or to Digital Credentials API presentment), and document conformance classes per primitive.

---

## 6. Practical On-Ramp for a Small Team

**IETF individual draft (cost: $0):**
1. Create a free Datatracker account; author in Markdown using kramdown-rfc or the IETF authoring tools; submit via datatracker.ietf.org/submit at any time (no membership, no fee) [1].
2. The draft is instantly public and citable ("I-D exists"); it expires after ~6 months unless refreshed (ARKG is on revision -10, which is normal cadence [11]).
3. Socialize on the relevant WG list (COSE, LAMPS, or OAuth) and request agenda time; remote meeting participation is possible (registration fees apply for meetings, with fee-waiver options; list participation is free).
4. IPR: by submitting you accept the Note Well; disclose any patents you hold that read on the draft.

**W3C Community Group (cost: $0):**
1. Create a free W3C account; propose the CG with a short scope statement; it launches once 5 people support it — no charter approval needed [12].
2. Every participant signs the Community Contributor License Agreement (copyright + patent commitments on their own contributions); when the spec stabilizes, publish it as a Final Report under the Final Specification Agreement to lock in royalty-free commitments [12].
3. CGs "do not create W3C standards" — the exit is transition to a WG (here: feeding WebAuthn L4 or a future rechartered group), for which the CG-to-WG path is explicitly designed [12].
4. Full W3C *membership* (needed only for WG voting, not CG work or GitHub contributions) has a fee scale by organization size and country [17]; WebAuthn WG accepts non-member technical submissions under the Patent Policy and can appoint Invited Experts [4].

**OpenID Foundation (cost: $50–$1,000):** join as an individual ($50) or small org ($1,000 for ≤25 employees), sign the per-WG IPR contribution agreement, and participate in DCP calls to keep the identity-binding layer aligned [13].

**Cloud Signature Consortium (cost: ~€2,250–3,000):** Associate membership grants the conformity checker and working-group access sufficient to propose a passkey-authorization profile for CSC API [14].

**Total realistic year-one standards budget for a small company: under €5,000** (CSC Associate + OIDF small-org + $0 for IETF and W3C CG), plus the real cost — roughly 0.2–0.5 FTE of sustained editorial and meeting participation. The FIDO2 and OIDC precedents both show that consistent individual editors, not corporate weight, are what move layered specs.

---

## Sources

1. IETF Datatracker, document search "passkey" (results: draft-sander-open-tabs-passkey-00; draft-bucksch-sasl-passkey-00) — https://datatracker.ietf.org/doc/search?name=passkey&rfcs=on&activedrafts=on&olddrafts=on
2. Cloud Signature Consortium, "Download API specifications" (CSC API v2.2, 6 Nov 2025; CSC Data Model v1.0.0, 21 Oct 2025; v2.1.0.1, Jan 2025; v2.0, Apr 2023) — https://cloudsignatureconsortium.org/resources/download-api-specifications/
3. W3C, Web Authentication Working Group Charter 2024–2026 (scope, out-of-scope, FIDO2 TWG coordination, Patent Policy 2020) — https://www.w3.org/2024/04/wg-webauthn-charter.html ; charter history — https://www.w3.org/groups/wg/webauthn/charters/
4. W3C, DRAFT Web Authentication WG Charter 2026–2028 (scope item 11 "Signing additional data…"; WebAuthn L3 CR 2026-01-13; L4 expected ~Q4 2028; WICA CG coordination) — https://w3c.github.io/charter-drafts/2026/charter-wg-webauthn-2026.html
5. w3c/webauthn PR #2078, "Add 'sign' extension" (E. Lundberg, opened 2024-05-28; open; milestone L4 FPWD; labels @Risk, FeatureProposal) — https://github.com/w3c/webauthn/pull/2078
6. FIDO Alliance, User Authentication Specifications overview (FIDO2 = WebAuthn + CTAP; UAF; U2F/CTAP1) — https://fidoalliance.org/specifications/
7. FIDO Alliance, Membership application (Membership Agreement, Bylaws, IPR Summary "v4 W3C") — https://fidoalliance.org/membership-application/ ; Digital Credentials focus area — https://fidoalliance.org/fido-alliance-digital-credentials/ ; Credential Exchange — https://fidoalliance.org/specifications-credential-exchange-specifications/
8. ETSI, TC ESI committee page (remit: signature formats, TSP requirements, remote signature creation, eIDAS support) — https://www.etsi.org/committee/esi
9. ETSI deliverables directory for TS 119 432 (V1.1.1 2019, V1.2.1 2020, V1.3.1 dated 2026-03-31) — https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/
10. OpenID Foundation, Digital Credentials Protocols (DCP) WG page (charter goals, format-agnostic scope, liaisons with EC/DIF/ETSI/ISO SC17, ISO 18013-7 & 23220 profiling OID4VP, security analyses 2023 & 2025, chairs) — https://openid.net/wg/digital-credentials-protocols/
11. IETF, draft-bradleylundberg-cfrg-arkg-10, "The Asynchronous Remote Key Generation (ARKG) algorithm" (active individual draft, updated 2026-02-27; repo github.com/Yubico/arkg-rfc) — https://datatracker.ietf.org/doc/draft-bradleylundberg-cfrg-arkg/
12. W3C, "About Community and Business Groups" (free, open, CLA, Final Specification Agreement, tuned for standards-track transition) — https://www.w3.org/community/about/
13. OpenID Foundation, "Join the OpenID Foundation" (fees: $50 individual / $250 gov-nonprofit / $1,000 ≤25 employees / $5,000 26–100 / $20,000 >100 / $50,000 sustaining; IPR policy link) — https://openid.net/foundation/benefits-members/ ; Digital Credentials Harmonized Presentation WG — https://openid.net/wg/digital-credentials-harmonized-presentation-working-group/
14. Cloud Signature Consortium, "Join Us" (Associate €3,000 / Premium €6,000 / Executive €10,000; 25% SME/non-profit discount) — https://cloudsignatureconsortium.org/join-us/ ; CSC report "EUID Wallet – How to implement Free QES" — https://cloudsignatureconsortium.org/csc-report-euid-wallet-how-to-implement-free-qes/
15. [reference] ISO 32000-2 (PDF 2.0) — https://www.iso.org/standard/75839.html
16. [reference] ISO/IEC 18013-5 (mDL) — https://www.iso.org/standard/69084.html
17. [reference] W3C membership fees — https://www.w3.org/membership/fees/
18. [reference] W3C, Secure Payment Confirmation (Web Payments WG; transaction-confirmation ceremony layered on WebAuthn) — https://www.w3.org/TR/secure-payment-confirmation/
19. GitHub API repository search "passkey signing" (top results are authentication libraries; no document-signing standard efforts) — https://api.github.com/search/repositories?q=passkey+signing&sort=stars
