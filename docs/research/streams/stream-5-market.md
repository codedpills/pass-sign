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
