# PassSign Document 3 — Legal & Regulatory Landscape

**Phase 2 deliverable (Stream 4: eIDAS primary; US, UK, Singapore, Australia comparative)**
Version 0.1 · 2026-07-17 · PassSign Research Program

*This document is regulatory research conducted to inform protocol design. It is not legal advice; conclusions on eIDAS classification, sole-control sufficiency, and enforceability require review by qualified counsel before any RFC publication or product claim.*

This document answers primary question **Q2 (Is it legally viable?)**. It consists of this synthesis followed by the full Stream 4 research document.

---

## Executive synthesis

### The legal ceiling, by authenticator class

| Construction | EU (eIDAS) | US/UK/AU | Singapore |
|---|---|---|---|
| Synced passkey, direct assertion | **SES** with best-in-class attribution evidence; process-based AdES claim arguable but unsettled | Strong attribution evidence under UETA §9 / ECA s.7 / ETA s.10 (no case law yet) | Plausible s.18(1) "secure electronic signature" as agreed security procedure |
| Device-bound passkey + attestation + identity binding | **Defensible AdES** (Art 26), provided identity layer fixes 26(a)/(b) and document hash is challenge-bound for 26(d) | Same, stronger | Same, stronger |
| Passkey as activation factor for QTSP-held key (remote QES) | **QES** — the passkey plays the SCAL2 signature-activation role | QES recognition varies; strongest evidence class regardless | Qualified-certificate regime available |

### The key legal lever: sole control ≠ key custody

EU law already accepts that "sole control with a high level of confidence" (Art 26(c)) is satisfied by *exclusive activation capability*, not physical key possession: the dominant QES model today is a QTSP-held key in a certified remote QSCD, activated per-signature via signature activation data under CEN EN 419 241-2 (SCAL2). A UV-required passkey assertion is an activation event of directly comparable structure. This precedent is the doctrinal bridge that makes the Phase 1 baseline architecture (passkey-activated remote signing key) not merely technically convenient but *legally load-bearing* — it slots into the exact pattern regulators already certify.

Conversely, synced passkeys face an unresolved question: key material replicated across devices through provider escrow, with no attestation and RP-invisible recovery events, has no authoritative ETSI/ENISA assessment against Art 26(c). This is a genuine gap in the regulatory landscape, not a settled negative — but it means AdES claims for synced passkeys carry unquantified risk until standards bodies or courts speak.

### What WebAuthn is missing legally is identity, not cryptography

Art 26(a)/(b) require the signature be uniquely linked to and capable of identifying *the signatory* — a natural person. WebAuthn identifies credentials, not persons. The gap between "passkey ceremony" and "advanced electronic signature" is closed by an identity-binding layer (verified identity bound to the credential and carried in the signature envelope), which is precisely what the Phase 1 composition provides via wallet credentials — plus tamper-evident document binding (challenge = document manifest hash) for 26(d).

### eIDAS 2.0 verified facts and their consequence

The EUDI Wallet must enable signing by QES "by default and free of charge" for non-professional use (Art 5a(5)(g), with Art 5a(4)(e)); rollout is uneven mid-2026 (under a third of Member States ready), and a deadline discrepancy exists between wallet availability (Dec 2026) and the CSC-stated free-QES date (Nov 2027) [unverified]. Consequence: **QES is being commoditized in the EU for consumer statutory-form use cases.** PassSign should not compete there; it should (a) supply the ceremony/evidence layer, (b) target the dominant SES/AdES volume market globally, and (c) define the QES on-ramp through QTSPs rather than rebuilding qualification.

### Market-requirement reality

The overwhelming bulk of commercial signing (NDAs, B2B contracts, HR, consumer T&Cs) is legally satisfied by SES; AdES is chosen for risk posture; QES is mandatory only for a statutory-form tail (e.g., German BGB §126a contexts). A protocol delivering *excellent SES and defensible AdES with a QES on-ramp* covers the commercially dominant share of real-world signing.

### Scorecard

| Question | Verdict | Basis |
|---|---|---|
| **Q2 — Legally viable?** | **Supported.** SES everywhere immediately; AdES defensible via device-bound profile + identity binding; QES reachable by composition with QTSP remote signing (passkey as SCAL2 activation factor). The decision-framework refutation condition ("no path above SES") does not hold. | Stream 4 §§1–8 |

### Design directives handed to Phase 4/5

1. Two assurance tiers as protocol profiles: synced-passkey (attribution/SES-grade) vs device-bound-attested (AdES-grade) — converting legal ambiguity into a product tier.
2. Authenticator class, identity-binding evidence, and consent/disclosure capture (ESIGN §101(c)) recorded as first-class ceremony attributes.
3. JAdES-wrapped evidence with qualified timestamps and LTV material so EU validators process PassSign outputs natively.
4. Standards-venue goal: formal recognition of WebAuthn UV assertions as an acceptable SCAL2 authentication mechanism (building on existing FIDO–eIDAS white papers).

---

*The full Stream 4 research document follows, including the requirement-by-requirement Art 26 analysis, jurisdiction comparisons, limitations register, and sources.*

---

# Stream 4: Legal & Regulatory Landscape

**PassSign Program — Research Stream 4 (Legal Frameworks)**
**Date:** 2026-07-17 | **Scope:** EU/eIDAS (primary); US, UK, Singapore, Australia (comparative)

> **Disclaimer.** This document is regulatory research intended to inform protocol design for the PassSign program. It is not legal advice, and no statement herein should be relied upon as a definitive legal, tax, or compliance opinion. Statute and standard citations were verified against primary or reputable secondary sources where possible; claims that could not be verified against a fetched source are marked **[unverified]**. Qualified counsel should review any conclusions before they inform product or compliance decisions.

---

## 1. eIDAS Tiers & eIDAS 2.0 Changes

### 1.1 The three-tier model (Regulation (EU) No 910/2014, as amended by Regulation (EU) 2024/1183)

| Tier | Definition | Legal effect |
|---|---|---|
| **SES** — (simple) electronic signature | Art 3(10): "data in electronic form which is attached to or logically associated with other data in electronic form and which is used by the signatory to sign" | Art 25(1): may not be denied legal effect or admissibility in legal proceedings *solely* because it is electronic or non-qualified (non-discrimination). Probative weight is for the court. |
| **AdES** — advanced electronic signature | Art 3(11), with requirements in Art 26 (see §2 below) | Same Art 25(1) non-discrimination baseline; no statutory equivalence to handwriting. Some sector rules and national laws specifically require AdES. |
| **QES** — qualified electronic signature | Art 3(12): an AdES that is (i) created by a **qualified signature creation device (QSCD)** meeting Annex II, and (ii) based on a **qualified certificate** meeting Annex I, issued by a qualified trust service provider (QTSP) | Art 25(2): "shall have the equivalent legal effect of a handwritten signature." Art 25(3): a QES based on a qualified certificate issued in one Member State "shall be recognised as a qualified electronic signature in all other Member States" (mutual recognition). |

The tiers form a strict pyramid: every QES is an AdES; every AdES is an SES. Only QES receives automatic handwritten-signature equivalence and EU-wide mutual recognition. AdES and SES remain valid and admissible, but the burden of demonstrating reliability in a dispute sits with the relying party; for QES, the qualified status effectively reverses the practical burden.

Consolidated text: EUR-Lex CELEX 02014R0910-20241018 [Source 1].

### 1.2 eIDAS 2.0 (Regulation (EU) 2024/1183) and the EUDI Wallet signing obligations

Regulation 2024/1183 (in force 20 May 2024) inserts Articles 5a–5f into Regulation 910/2014, creating the **European Digital Identity (EUDI) Wallet**. The wallet-and-QES provisions were verified against the full Article 5a text [Source 2]:

- **Art 5a(1):** each Member State shall provide at least one EUDI Wallet within **24 months of entry into force of the implementing acts** under Art 5a(23) and Art 5c(6). The first batch of implementing acts was adopted in late November 2024, yielding the widely cited availability deadline of **December 2026** (commonly stated as 24 December 2026) [Sources 8, 9]. Note: the Cloud Signature Consortium refers to a **November 2027** deadline "by which all Member States must offer at least one EUDI Wallet with free qualified e-signatures for non-professional use" [Source 6] — the divergence presumably reflects different implementing-act entry-into-force dates for the signing functionality; treat the exact free-QES deadline as **[unverified — Dec 2026 vs Nov 2027 discrepancy between sources]**.
- **Art 5a(4)(e):** wallets shall enable the user to "**sign by means of qualified electronic signatures** or seal by means of qualified electronic seals."
- **Art 5a(5), first subparagraph, point (g):** wallets shall "**offer all natural persons the ability to sign by means of qualified electronic signatures by default and free of charge**."
- **Art 5a(5), second subparagraph:** "Notwithstanding point (g)…, Member States may provide for proportionate measures to ensure that the use of qualified electronic signatures free-of-charge by natural persons is **limited to non-professional purposes**." The Regulation does not define "non-professional," leaving Member State divergence likely [Source 5].
- **Art 5a(5)(a)(xi):** wallets shall support common protocols and interfaces "for the creation of qualified electronic signatures or electronic seals by means of qualified electronic signature or electronic seal creation devices" — i.e., the wallet is an interface to a QSCD, which may be local or remote.
- **Art 5a(11):** wallets are provided under an eID scheme at assurance level **high**; Art 5a(5)(d) applies Level-of-Assurance-high requirements to identity proofing and authentication.

**Wallet-as-QSCD path.** The Regulation deliberately does not mandate where the signing key lives. Two architectures are contemplated in the ecosystem: (a) the wallet's secure hardware (phone secure element / eSIM / external token) is itself certified as a QSCD; (b) the wallet acts as the authentication and signature-activation channel to a **remote QSCD operated by a QTSP** (remote QES). Industry consensus, including the CSC white paper on the wallet as "a new tool for remote signing" [Source 7], is that the remote model will dominate at launch because certifying heterogeneous consumer handsets as QSCDs is slow and fragile (see §3).

Also relevant: eIDAS 2.0 elevates the operation of a remote QSCD to a **qualified trust service in its own right** (management of remote qualified signature creation devices, new Art 29a/Annex points), whereas under eIDAS 1.0 it was an adjunct activity of a QTSP [Source 10].

---

## 2. AdES Article 26 vs Passkeys — Requirement by Requirement

Art 26 requires an AdES to be:

> (a) uniquely linked to the signatory; (b) capable of identifying the signatory; (c) created using electronic signature creation data that the signatory can, with a high level of confidence, use under his sole control; and (d) linked to the data signed therewith in such a way that any subsequent change in the data is detectable.

Assessment of a WebAuthn/passkey signing ceremony (user-verification-required assertion over a document hash bound into the challenge):

### (a) Uniquely linked to the signatory

A passkey key pair is unique per (relying party, user account). The private key is statistically unique and never shared across users. **However**, uniqueness of the *credential* is not automatically uniqueness to the *natural person*: passkey registration binds a key to an *account*, and the account-to-person link depends on the identity proofing done at enrollment. A protocol that wants AdES-grade "unique linkage" must pair the passkey with an identity-proofing event (or a certificate/attestation of attributes) and record that binding. **Verdict: satisfiable by design; not inherent in WebAuthn alone.**

### (b) Capable of identifying the signatory

Same dependency: WebAuthn identifies a credential (credential ID, user handle), not a legal identity. PKI-based AdES meets (b) via the certificate's subject DN. A passkey scheme meets (b) only if the signing record links the credential to verified identity data (IDV session, eID, qualified certificate, or wallet attestation). **Verdict: satisfiable via an identity-binding layer; the FIDO–eIDAS white papers reach the same conclusion — FIDO supplies authentication strength, not identification semantics [Sources 11, 12].**

### (c) Sole control "with a high level of confidence"

This is the decisive requirement and the one where **synced vs device-bound passkeys diverge**:

- **Device-bound passkeys** (security key or platform authenticator with the key confined to a secure element, attestation available): private key never leaves certified-or-certifiable hardware; user verification (biometric/PIN) gates each use; attestation lets the relying party verify authenticator model and key protection at registration. This is materially the same control model as a smartcard — the historical archetype of "sole control." The FIDO Alliance/eIDAS white papers argue FIDO2 hardware authenticators meet eID assurance "substantial/high" and can anchor sole control for remote signing [Sources 11, 12]. **Verdict: strong case for satisfying (c).**
- **Synced passkeys** (iCloud Keychain, Google Password Manager, third-party managers): the private key is generated locally, encrypted, and replicated through the provider's cloud sync fabric to all devices in the user's account; **no attestation is available** for synced passkeys, so a relying party cannot cryptographically verify how the key is protected [Source 13]. Three concerns for (c): (i) key material exists, in encrypted form, outside any single device, with recovery flows (e.g., account recovery, new-device enrollment) controlled by the sync provider and gated by account credentials that may be weaker than the passkey itself; (ii) family/credential-sharing features on some platforms permit deliberate sharing of passkeys, directly undermining exclusivity; (iii) without attestation, "high level of confidence" cannot be demonstrated to a verifier — only asserted. Published legal/technical commentary directly analyzing synced passkeys against Art 26(c) is sparse; searches surfaced the technical synced-vs-device-bound distinction and the discontinuation of the devicePubKey/supplementalPubKeys extensions that would have provided device-level trust signals for synced passkeys [Source 13], but **no authoritative ETSI/ENISA position on synced passkeys and Art 26(c) was found [unverified — apparent gap in the literature rather than a settled negative]**.

**The remote-QES precedent cuts in favor of passkeys.** EU standardization has already accepted that **sole control does not require physical possession of the key**. Under CEN EN 419 241-1/-2, a QTSP holds and operates the signing key in a certified remote QSCD, and the signatory's sole control is established through the **Signature Activation Protocol (SAP)** carrying **Signature Activation Data (SAD)** at **Sole Control Assurance Level 2 (SCAL2)** — i.e., control is exercised via strong, signer-exclusive *authorization* of each signature, not via custody of the key [Sources 14, 15, 16]. By analogy: a passkey ceremony in which the key sits in a provider-managed enclave, is unusable without per-operation user verification, and produces an auditable, origin-bound, per-document authorization is structurally similar to SAD-based activation. The counterargument is that SCAL2 rests on Common Criteria-certified modules and audited QTSP operations, whereas consumer sync fabrics are uncertified and their recovery paths are not designed to SCAL2 threat models. So the analogy supports the *concept* (sole control ≈ exclusive activation capability) but not automatic *equivalence of assurance*.

**Practical verdicts on (c):**
- Device-bound passkey + attestation: defensible AdES-grade sole control; also the plausible substrate for local QSCD certification (see §3).
- Synced passkey: arguable SES-plus / low-end-AdES. A litigant could credibly attack "high level of confidence" via sync-fabric recovery flows and lack of attestation. Best treated as **strong attribution evidence** rather than a guaranteed AdES ingredient; an AdES claim would rest on the *overall process* (signing service controls, session identity verification, audit trail), which is how DocuSign-style AdES products already argue their case.

### (d) Linked so that subsequent change is detectable

WebAuthn signs `authenticatorData || SHA-256(clientDataJSON)`, and the challenge inside clientDataJSON can embed the document hash. If the protocol binds the full document (or a signed-data envelope per ETSI AdES formats — XAdES/PAdES/CAdES/JAdES) into the challenge, any alteration invalidates verification. **Verdict: fully satisfiable; this is a protocol-design task (canonical hashing, challenge construction, timestamping), not a legal obstacle.** Note that WebAuthn signatures are raw ECDSA/RSA assertions, not ETSI AdES container formats; wrapping the assertion in a JAdES/PAdES-conformant structure (Stream 3's problem) strengthens the Art 26(d) and long-term-validation story.

---

## 3. QSCD Certification & the Remote QES Model

### 3.1 Certification path

Annex II of eIDAS sets QSCD requirements (confidentiality of creation data, practical uniqueness, non-derivability, protection against forgery and against use by others; the device must not alter the data to be signed). Certification routes (Art 30, Art 31, Commission Implementing Decision (EU) 2016/650):

- **Local QSCDs:** Common Criteria evaluation against protection profiles in the **EN 419 211** series (secure signature creation devices) — smartcards, USB tokens, some eSIM/SE configurations.
- **Remote QSCDs:** Common Criteria against **CEN EN 419 241-2** (Protection Profile for QSCD for Server Signing) together with **EN 419 221-5** (Cryptographic Module for Trust Services), supported by ETSI TS 119 431-1/-2 (policy requirements for TSPs operating remote QSCDs / supporting AdES creation) and TS 119 432 (remote signature creation protocols) [Sources 14, 15, 16]. Member State designated bodies may also certify via alternative processes using "comparable security levels" (Art 30(3)(b)), which some national schemes have used **[unverified detail]**.

### 3.2 Can a consumer phone secure element be a QSCD?

In principle yes; in practice rarely and not at fleet scale. Obstacles: (i) Common Criteria certification attaches to a specific hardware/software configuration — consumer handsets ship in thousands of configurations with frequent OS updates, so a certificate is obsolete or scope-limited quickly; (ii) the phone SE/StrongBox/Secure Enclave is certified (if at all) for other schemes (e.g., EMVCo, CC EAL for the chip) rather than against EN 419 211 profiles; (iii) the QSCD status must be certifiable *to the supervisory body*, and attestation from synced-passkey ecosystems is absent. There are national examples of mobile-based QSCDs (e.g., SIM-based schemes and wallet pilots exploring local WSCD certification) **[unverified — specific certified consumer-phone QSCDs not confirmed in this research pass]**. The EUDI architecture reference framework anticipates external tokens, eSIM/eUICC, and remote HSMs as Wallet Secure Cryptographic Devices, with remote HSM the pragmatic default **[unverified detail of ARF versions]**.

### 3.3 The remote QES model — dominant today

The dominant production QES model in the EU: the **QTSP generates and holds the user's key in a certified remote QSCD (HSM)**; the user authorizes each signature through a SAM (Signature Activation Module) via SAD at SCAL2 [Sources 14, 15, 16]. eIDAS 2.0 makes remote-QSCD management a distinct qualified trust service [Source 10]. A December 2024 revision, **ETSI TS 119 431-1 v1.3.1**, further streamlines one-shot signing: for **one-time (short-lived) certificates**, identification of the user once within a single bounded session suffices for both certificate issuance and signature creation — no separate second authentication (OTP) required [Source 16]. This matters for PassSign: EU standardization is actively lowering the ceremony cost of QES, and a passkey is an obvious candidate to *be* the identification/authentication event in such flows.

**Key implication:** in the remote model, the *authentication mechanism that conveys the SAD* is exactly where a passkey slots in. FIDO Alliance white papers explicitly propose FIDO2 as the sole-control authentication mechanism for QTSP remote signing [Sources 11, 12]. A passkey does not need to hold the qualified signing key to participate in QES — it needs to be an acceptable SCAL2 activation factor.

---

## 4. United States: ESIGN & UETA

### 4.1 Framework

- **ESIGN Act, 15 U.S.C. § 7001(a)** [Source 17]: a signature, contract, or record "may not be denied legal effect, validity, or enforceability solely because it is in electronic form." Technology-neutral; no tiers.
- **UETA (1999)**, enacted in 49 states + DC (New York has its own ESRA): §2(8) defines electronic signature as "an electronic sound, symbol, or process attached to or logically associated with a record and executed or adopted by a person with the **intent** to sign"; §7 parallels ESIGN's non-discrimination [Source 18].
- **Attribution — UETA §9(a):** an electronic signature "is attributable to a person if it was **the act of the person**. The act of the person may be shown **in any manner, including a showing of the efficacy of any security procedure** applied to determine the person to which the electronic record or electronic signature was attributable." §9(b): the effect of attribution is determined from context and surrounding circumstances [Source 18].
- **Consumer consent — ESIGN § 101(c) (15 U.S.C. § 7001(c)):** where a law requires information to be provided to a consumer in writing, an electronic record suffices only if the consumer affirmatively consents after receiving prescribed disclosures and consents (or confirms consent) "in a manner that reasonably demonstrates that the consumer can access information in the electronic form" used [Source 17].
- **Retention — ESIGN § 101(d) / UETA §12:** record-retention requirements are met by an electronic record that accurately reflects the information and **remains accessible** to entitled persons for later reference.

### 4.2 What attribution evidence do courts weigh?

Because §9 permits proof "in any manner," litigation over e-signatures is evidentiary, not doctrinal. Courts have weighed: authenticated account credentials and password-protected portals; audit trails (IP address, timestamps, email routing, device data); the surrounding course of conduct (payment, performance, non-repudiation of related steps); and the efficacy of the vendor's security procedure. In *Zulkiewski v. American General Life Ins. Co.* (Mich. Ct. App. 2012), the court confirmed that UETA does **not** require proof of a security procedure's efficacy — it is one available route among "any manner" of showing the act of the person [Source 19]. Conversely, attribution has failed where credentials were shared among household members or the proponent could not connect the click to the defendant (a recurring pattern in employment-arbitration cases) **[unverified — pattern well established in practice literature; specific case list not re-verified this pass]**.

### 4.3 Would a passkey ceremony be strong attribution evidence?

Yes — plausibly the strongest commodity evidence yet available under §9:

- **User verification:** each assertion requires biometric or PIN verification on the authenticator; the UV flag is recorded in signed authenticator data. This directly answers "was it the act of the person," the exact question §9 asks — far better than a password (shareable, phishable) or an emailed signing link (attributes to an inbox).
- **Origin binding:** WebAuthn signatures are cryptographically bound to the relying-party ID, defeating phishing-based repudiation narratives ("I signed something, but on a fake site").
- **Per-credential asymmetric proof:** the signature verifies against a public key registered to the specific account, with a signature counter and flags — a self-authenticating audit artifact.
- **Attestation (device-bound only):** adds provable statements about the authenticator's key-protection class.

Caveats: (i) a synced passkey shared via platform sharing features, or a family member enrolled in the device biometrics, re-opens the "someone else's finger" argument — mitigations are procedural (attest UV, record enrollment context, contemporaneous identity checks for high-value signings); (ii) §9 attribution still requires linking the *account* to the *person* at enrollment, so onboarding evidence remains part of the record; (iii) no reported US decision squarely evaluating WebAuthn/passkey evidence for signature attribution was located **[unverified — likely none yet exists]**. For US purposes, synced vs device-bound matters far less than in the EU: both comfortably exceed prevailing evidentiary practice.

---

## 5. United Kingdom

- **Electronic Communications Act 2000, s.7:** electronic signatures and supporting certificates are **admissible in evidence** in relation to questions of authenticity or integrity of an electronic communication. English law's signature concept is otherwise common-law and famously liberal (an "X," a typed name, a click can suffice given intent and any required formality).
- **Retained/assimilated UK eIDAS** (Regulation 910/2014 as retained by the EU (Withdrawal) Act 2018 and amended by SI 2019/89): keeps the SES/AdES/QES taxonomy and Art 25-style effects in UK law, but UK QES status requires a UK-recognised QTSP. In practice **there are no QTSPs on the UK trusted list; the market relies on EU-certified QTSPs**, which the UK framework accommodates [Source 20]. The **Data (Use and Access) Act 2025, Part 7 (Trust services)** amends UK eIDAS, including enabling recognition of EU conformity assessment so EU QTSPs can operate under the UK regime without disproportionate barriers [Sources 20, 21]. A government call for views on trust-services adoption closed 20 September 2025; DSIT published follow-up work in 2026 [Source 22].
- **Law Commission, *Electronic Execution of Documents* (Law Com No 386, 2019):** authoritative statement that an electronic signature is **capable in law of executing a document (including a deed)** provided the signatory intends to authenticate and applicable formalities (e.g., witnessed attestation for deeds, with the witness physically present) are satisfied; no tier requirement — QES is *not* needed for validity [Source 23 — conclusions widely reproduced; original report not re-fetched this pass, treat exact wording as **[unverified]**].
- **Industry Working Group on Electronic Execution** (established on the Law Commission's recommendation): interim report (Feb 2022) with best-practice guidance for electronic execution, and a final report addressing deeds and optimal execution practices [Source 22; details **[unverified]**].

**UK ceiling for PassSign:** validity is essentially never the issue; evidence is. A passkey ceremony functions as high-grade evidence of authenticity under s.7 and common law. UK QES equivalence exists on paper but is commercially marginal today.

---

## 6. Singapore & Australia (brief)

### Singapore — Electronic Transactions Act 2010 (2020 Rev. Ed.)

- Baseline validity: s.8 (signature requirement satisfied by a method identifying the person and indicating intention, reliable as appropriate or proven in fact).
- **Secure electronic signature — s.18(1):** through a specified or commercially reasonable security procedure agreed by the parties, it can be verified that at signing time the signature was: (a) **unique** to the person; (b) **capable of identifying** the person; (c) created in a manner or using a means under the **sole control** of the person; and (d) **linked** to the record such that alteration invalidates the signature [Source 24]. This is essentially Art 26 with an added "security procedure" wrapper.
- **Presumptions — s.19:** a secure electronic signature is presumed (rebuttable) to be the signature of the person to whom it correlates, and to have been affixed with intent to sign/approve [Source 24]. Singapore thus offers a statutory *evidentiary presumption* — the sharpest legal prize among the common-law jurisdictions surveyed. A passkey ceremony framed as a "commercially reasonable security procedure agreed to by the parties" is a credible route to secure-signature status; the sole-control analysis mirrors the eIDAS discussion (device-bound comfortably; synced arguably). First Schedule exclusions (e.g., wills, certain property/trust instruments) persist **[unverified — current schedule contents post-2021 amendments not re-checked]**.

### Australia — Electronic Transactions Act 1999 (Cth)

- **s.10(1):** an electronic signature satisfies a Commonwealth-law signature requirement if (a) a **method identifies the person and indicates their intention**; (b) the method was **as reliable as appropriate** for the purpose, or proven in fact to have identified/indicated intention; and (c) the other party **consents** [Source 25 — text well established; not re-fetched, **[unverified verbatim]**]. Mirrored by State/Territory ETAs; some instrument classes are exempted by regulation. No tiers, no presumptions. Any competent passkey ceremony clears this bar; the Australian analysis is purely evidentiary, as in the US.

---

## 7. Tier Requirements by Document Type (EU focus)

Where each tier is required is largely a matter of **national** law layered on eIDAS (Art 2(3): eIDAS does not touch national form requirements):

- **QES-mandatory (illustrative):** wherever national law equates statutory "written form" with QES — e.g., German BGB §126a (electronic form replacing written form requires QES), consumer-credit and suretyship contexts in several Member States; certain court filings and public-sector submissions; some corporate/notarial filings; EU-level examples include certain public procurement (ESPD/tender signing per Directive 2014/24/EU implementations) **[unverified — country-by-country list not exhaustively verified]**.
- **AdES-appropriate:** regulated financial services onboarding in some Member States, some employment documents, insurance; often chosen for risk posture rather than strict mandate.
- **SES-sufficient:** the overwhelming bulk of commercial life — NDAs, most B2B commercial contracts, procurement outside statutory form, HR routine documents, invoices, consumer T&Cs. **Practical dominance:** DocuSign/Adobe-style SES (click-to-sign plus audit trail, sometimes marketed with an "AdES" wrapper via provider-held certificates) is the de facto standard across the EU, US, UK, SG, AU; QES volume, while growing (notably in DACH, Baltics, Southern Europe), remains a minority share **[unverified — market-share figures belong to Stream 5]**.

Design consequence: a protocol that maxes out at "excellent SES / defensible AdES" already covers the dominant share of real-world signing; QES coverage is needed only for the statutory-form tail — which the EUDI Wallet is about to commoditize anyway (§8).

---

## 8. EUDI Wallet QES Delivery Model & Impact

**Delivery model.** The consensus architecture for wallet QES is **remote QES via a QTSP**: the qualified certificate and key are created/held in the QTSP's remote QSCD; the wallet performs LoA-high user authentication and conveys the signature activation (SAD) — i.e., the wallet is the *authentication and authorization channel*, not (initially) the QSCD itself [Sources 6, 7]. The CSC report "EUDI Wallet – How to implement Free QES" (Sept 2025) outlines **two implementation models for the free-QES obligation: a fully public infrastructure, or a public–private partnership** with commercial QTSPs, and frames the open question of who bears the cost of "free" [Source 6; the underlying PDF is described as not for public distribution — contents beyond the published abstract **[unverified]**].

**Status (mid-2026).** Member States face the December 2026 availability deadline; per July 2026 tracking, rollout is "visible but uneven": Denmark launched a production wallet (AltID, 3 June 2026); Germany runs sandboxes with public rollout expected early 2027; France's France Identité is not yet certified as an EUDI Wallet; fewer than one-third of Member States met a readiness benchmark, and the Commission has expressed doubt that all will launch on time [Sources 8, 9, 26]. Certification per Art 5c (cybersecurity schemes under the Cybersecurity Act plus national schemes) is itself a bottleneck. Private relying parties subject to strong-customer-authentication obligations must accept the wallet within 36 months of the implementing acts (Art 5f(2)).

**Implication for a complementary open protocol.** Free wallet QES will, over 2027–2030, compress the price of the *qualified* tier toward zero for EU natural persons in non-professional contexts — but through a heavyweight, Member-State-mediated, QTSP-centric stack. What it does **not** provide: (i) coverage outside the EU/EEA; (ii) professional/corporate signing (explicitly carve-out-able from free QES); (iii) a lightweight web-native ceremony for the enormous SES/AdES majority of documents; (iv) relying-party-side developer ergonomics comparable to WebAuthn. A passkey-native protocol is therefore best positioned as **complementary**: the universal SES/AdES layer with a defined on-ramp to QES — either by acting as the SCAL2 activation factor for QTSP remote signing (per the FIDO-QTSP pattern and the streamlined TS 119 431-1 v1.3.1 one-time-certificate flow), or by interoperating with wallet-initiated QES for the statutory-form tail.

---

## 9. Limitations Register

| # | Limitation | Impact |
|---|---|---|
| L1 | No authoritative ETSI/ENISA/regulator position located on **synced passkeys vs Art 26(c) sole control**; conclusion rests on analysis-by-analogy (SCAL2) and technical facts (no attestation for synced passkeys). | AdES claims for synced passkeys carry unquantified legal risk until standards bodies or courts speak. |
| L2 | Free-QES deadline discrepancy: Dec 2026 (general wallet availability) vs Nov 2027 (CSC's stated free-QES deadline). Exact obligation date **[unverified]**. | Affects timing assumptions for QES commoditization. |
| L3 | CSC free-QES report PDF contents beyond the public abstract not reviewed (marked not for public distribution). | Implementation-model detail is second-hand. |
| L4 | No reported US/UK case law found applying WebAuthn/passkey evidence to signature attribution. | "Strong attribution evidence" is a prediction, not precedent. |
| L5 | Country-by-country QES-mandatory document lists, Singapore First Schedule current text, Australia s.10 verbatim text, and Law Commission 2019 exact wording were not re-fetched this pass. | Marked [unverified]; low risk (well-established), but verify before publication. |
| L6 | Consolidated eIDAS text was not fetched directly from EUR-Lex this pass; Article texts verified via a secondary full-text mirror [Source 2] and standards literature. | Cross-check Annex I/II wording against EUR-Lex before final synthesis. |
| L7 | QSCD certification of consumer phone secure elements: no confirmed example of a certified consumer-handset QSCD found; ARF version specifics unverified. | Local-QSCD path assessment is directional. |

---

## 10. Legal Positioning Opportunities for PassSign

1. **Design to the evidentiary maximum, claim tiers honestly.** Position PassSign signatures as: (a) US/UK/AU — best-in-class attribution evidence under UETA §9 / ECA s.7 / ETA s.10 (user verification flag, origin binding, per-credential asymmetric proof, document-hash binding); (b) Singapore — engineered to satisfy the four s.18(1) criteria as an agreed "commercially reasonable security procedure," targeting the s.19 presumptions; (c) EU — SES always, **AdES by process** (identity-binding + audit + tamper-evident envelope), with device-bound-passkey mode as the high-assurance AdES profile.
2. **Split the profile: synced vs device-bound.** Make authenticator class a first-class, recorded, attestable protocol attribute. Synced-passkey signatures = "PassSign Standard" (SES/attribution-grade); device-bound-with-attestation = "PassSign Assured" (AdES-grade sole-control claim). This converts the legal ambiguity (L1) into a product tier rather than a protocol-wide ceiling.
3. **Exploit the SCAL2 precedent.** Argue — in standards venues, not only marketing — that EU law already accepts sole control as *exclusive activation capability* (EN 419 241-2 SAD/SCAL2), not key custody; a UV-required passkey assertion is an activation event of comparable structure. Target: an ETSI/CEN recognition of FIDO/WebAuthn as an acceptable SCAL2 authentication mechanism (building on the existing FIDO–eIDAS white papers), which would give passkeys a formal role inside QES rather than beneath it.
4. **Build the QES on-ramp, don't rebuild QES.** Specify a binding from a PassSign ceremony to (a) QTSP remote signing (CSC API / ETSI TS 119 432), including the TS 119 431-1 v1.3.1 one-time-certificate flow where a single identification event covers issuance + signing, and (b) EUDI Wallet-initiated QES. PassSign supplies the ceremony and evidence container; the QTSP supplies qualification.
5. **Fix Art 26(a)/(b) with an identity-binding layer.** Define how a verified identity (IDV, eID, wallet attestation, qualified certificate) is bound to a passkey credential and carried in the signature envelope — this, not cryptography, is the gap between WebAuthn and AdES.
6. **Wrap assertions in ETSI AdES containers** (JAdES is the natural fit for WebAuthn's JSON artifacts) with qualified timestamps and LTV material, so EU verifiers can process PassSign outputs with existing validation tooling and the Art 26(d) story is airtight.
7. **ESIGN consumer-consent module.** For US consumer documents, bake §101(c) disclosure/consent capture into the ceremony metadata to keep downstream records enforceable.
8. **Watch the free-QES rollout, position as complement.** As free wallet QES arrives (2026–2027+), the differentiators for PassSign are global reach, professional/B2B signing (outside the free-QES mandate), developer ergonomics, and the SES/AdES volume market that QES infrastructure over-serves.

---

## Sources

1. Consolidated eIDAS Regulation (EU) No 910/2014 (as amended by Reg. (EU) 2024/1183), EUR-Lex CELEX 02014R0910-20241018 — https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:02014R0910-20241018 *(not fetched this pass; article texts verified via Source 2)*
2. Regulation (EU) 2024/1183, Articles 5a–5f full text — https://www.european-digital-identity-regulation.com/Article_5a_(Regulation_EU_2024_1183).html *(fetched)*
3. Kennedys Law, "The European Digital Identity Framework" (2026) — https://www.kennedyslaw.com/en/thought-leadership/article/2026/the-european-digital-identity-framework-introducing-the-new-eu-digital-identity-wallet/
4. walt.id, "What Is the EUDI Wallet?" — https://walt.id/eidas2/eudi-wallet
5. Yousign, "eIDAS 2.0 Digital Identity Wallet: Compliance 2026" — https://yousign.com/blog/eidas-2-0-digital-identity-wallet-compliance-requirements
6. Cloud Signature Consortium, "CSC Report: EUID Wallet – How to implement Free QES" (Sept 2025) — https://cloudsignatureconsortium.org/csc-report-euid-wallet-how-to-implement-free-qes/ *(fetched; landing page and abstract)*
7. Cloud Signature Consortium, White Paper "The EU digital identity wallet: a new tool for remote signing" (Oct 2024) — https://cloudsignatureconsortium.org/wp-content/uploads/2024/10/CSC-White-Paper-The-EU-digital-identity-wallet-a-new-tool-for-remote-signing-with-qualified-electronic-signatures.pdf
8. eIDEasy, "EU Digital Identity Wallet Rollout Status by Member State (July 2026)" — https://www.eideasy.com/blog/eu-digital-identity-wallets-july-2026
9. Biometric Update, "EU Commission doubtful all member states will be able launch EUDI wallets this year" (Apr 2026) — https://www.biometricupdate.com/202604/eu-commission-doubtful-all-member-states-will-be-able-launch-eudi-wallets-this-year
10. Biometric Update / S. Elfors (IDnow), "New ETSI standard allows for remote signing with identification only" (Feb 2025) — https://www.biometricupdate.com/202502/new-etsi-standard-allows-for-remote-signing-with-identification-only *(fetched; also source for remote-QSCD-as-qualified-trust-service under eIDAS 2.0 and TS 119 431-1 v1.3.1)*
11. FIDO Alliance, "Using FIDO with eIDAS Services" White Paper (2020) — https://fidoalliance.org/wp-content/uploads/2020/06/FIDO_Using-FIDO-with-eIDAS-Services-White-Paper.pdf
12. FIDO Alliance, "Deploying FIDO2 for eIDAS QTSPs and eID schemes" White Paper (2020) — https://fidoalliance.org/wp-content/uploads/2020/04/FIDO-deploying-FIDO2-eIDAS-QTSPs-eID-schemes-white-paper.pdf
13. Corbado, "Digital Credentials & Passkeys: How they align & differ" (synced passkeys lack attestation; devicePubKey/supplementalPubKeys discontinued) — https://www.corbado.com/blog/digital-credentials-passkeys
14. CEN EN 419241-2:2019, "Trustworthy Systems Supporting Server Signing — Part 2: Protection profile for QSCD for Server Signing" — https://standards.iteh.ai/catalog/standards/cen/6161a882-7bd0-4450-a2ca-bf20251d6382/en-419241-2-2019
15. Entrust, "Signature Activation Module" white paper (SAP/SAD/SCAL2 framework) — https://www.entrust.com/sites/default/files/documentation/service-descriptions/signature-activation-module-sd.pdf
16. ETSI TS 119 431-1 (v1.3.1, Dec 2024) and TS 119 432, remote signing policy/protocol standards — summarized in Source 10
17. ESIGN Act, 15 U.S.C. § 7001 — https://uscode.house.gov/view.xhtml?req=granuleid:USC-prelim-title15-section7001&num=0&edition=prelim
18. Uniform Electronic Transactions Act (1999), official text with comments — https://euro.ecom.cmu.edu//program/law/08-732/Transactions/ueta.pdf (Uniform Law Commission: https://www.uniformlaws.org)
19. Proskauer, "Michigan Court Assesses Electronic Signature Authentication under UETA" (*Zulkiewski v. Am. Gen. Life Ins. Co.*, Mich. Ct. App. 2012) — https://newmedialaw.proskauer.com/2012/07/30/michigan-court-assesses-electronic-signature-authentication-under-uniform-electronic-transactions-act-in-online-insurance-transaction/
20. UK Government, "Adoption of trust services: call for views" and government response — https://www.gov.uk/government/calls-for-evidence/adoption-of-trust-services-call-for-views/adoption-of-trust-services-call-for-views
21. Data (Use and Access) Act 2025, Part 7 (Trust services) — https://www.legislation.gov.uk/ukpga/2025/18/part/7/crossheading/trust-services
22. DSIT blog, "Enhancing digital trust through trust services – an update" (June 2026) — https://enablingdigitalidentity.blog.gov.uk/2026/06/19/enhancing-digital-trust-through-trust-services-an-update/
23. Law Commission, *Electronic Execution of Documents* (Law Com No 386, 2019) — https://www.lawcom.gov.uk/project/electronic-execution-of-documents/ *(conclusions as widely reported; report not re-fetched)*
24. Singapore Electronic Transactions Act 2010, ss. 8, 17, 18, 19 — https://sso.agc.gov.sg/act/eta2010
25. Australia Electronic Transactions Act 1999 (Cth), s.10 — https://www.legislation.gov.au/Series/C2004A00553
26. Corbado, "EUDI Wallet 2026: Deadline, Rollout and Lessons" — https://www.corbado.com/blog/eudi-wallet-2026-deadline-rollout-eic-2026
