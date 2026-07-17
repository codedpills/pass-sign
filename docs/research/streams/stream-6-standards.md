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
