# PassSign Research Program — Execution Plan

**Version:** 0.1 (for review) · **Date:** 2026-07-14 · **Owner:** Zak
**Scope decisions:** Substantive-but-lean deliverables (10–25 pp each) · eIDAS-primary legal analysis · Markdown outputs · Review gate after each phase

---

## 1. Hypothesis under test

> A universal, passkey-native signing protocol can be created by combining WebAuthn, Verifiable Credentials, existing digital signature standards, and established legal trust frameworks into a single open protocol that provides a significantly better user and developer experience than current electronic signing solutions.

**Central framing question:** Is PassSign a genuinely new protocol, or a novel composition of existing standards? Every phase produces evidence for one side. The Gap Analysis (Phase 4) renders the verdict explicitly.

### Decision framework

Each phase closes by scoring its findings against the five primary questions:

| # | Question | Supported if… | Refuted if… |
|---|----------|---------------|-------------|
| 1 | Technically possible? | A signing ceremony can be built on WebAuthn primitives (natively or via a defined extension) that binds a user gesture to a document hash | Browser/authenticator constraints make document-bound signing architecturally impossible without replacing WebAuthn |
| 2 | Legally viable? | The construction maps to at least AdES under eIDAS and satisfies ESIGN/UETA intent + attribution | No path above "simple electronic signature" exists and market analysis shows SES-only is insufficient |
| 3 | Interoperable? | Output validates in existing PDF/CMS verification tooling, or a standard profile can make it so | Verification requires proprietary infrastructure end-to-end |
| 4 | Sufficiently differentiated? | Identified gaps in incumbents map to capabilities PassSign uniquely composes | Incumbents or in-flight standards (EUDI Wallet QES, CSC API) already deliver the same UX/DX |
| 5 | What's missing? | Missing pieces are specifiable (protocol work), not inventable (new cryptography) | Missing pieces require novel cryptographic primitives or unavailable platform APIs |

---

## 2. Phase structure

Seven streams are grouped into six phases. Streams 1–3 share deep technical dependencies (they all end at "what bytes can be signed and embedded where"), so they run together. Streams 5–6 share sources and framing, so they pair. Each phase ends with a review gate: findings summary + hypothesis scorecard + go/adjust decision from you.

```
Phase 1: Technology foundations   (Streams 1+2+3) → Doc 2: Technology Survey
Phase 2: Legal & regulatory       (Stream 4)      → Doc 3: Legal Landscape
Phase 3: Market & standards       (Streams 5+6)   → Doc 1: Current State Analysis
Phase 4: Gap analysis             (synthesis)     → Doc 4: Gap Analysis  ← verdict
Phase 5: Protocol design          (Stream 7)      → Doc 5: Protocol Proposal
Phase 6: RFC drafting                             → Doc 6: RFC Draft v0.1
```

Phases 2 and 3 have no dependency on each other and can be reordered or parallelized if you prefer; Phase 1 goes first because its findings determine which legal tiers and market segments are even reachable.

---

## Phase 1 — Technology Foundations (Streams 1–3)

**Goal:** Establish precisely what can be signed, by whom, with what keys, embedded in what container — the technical feasibility floor for the entire program.

### Stream 1: WebAuthn / Passkeys

Research questions (from your program, plus crux issues identified upfront):

- WebAuthn L2/L3 architecture: registration, assertion, RP model, browser mediation
- **What exactly is signed:** `authenticatorData || SHA-256(clientDataJSON)` — the critical constraint. Can arbitrary data (a document hash) be injected into the signed payload? Via `challenge`? What does that mean semantically and legally?
- Supported algorithms (COSE: ES256, RS256, EdDSA…) and their overlap with signature-container requirements (Stream 3)
- Extensions relevant to signing: `largeBlob`, `prf`, credential attestation; status of any proposed "sign" / arbitrary-payload extensions in W3C WebAuthn WG and FIDO2 CTAP
- **Synced passkeys vs. device-bound:** attestation is typically dropped for synced passkeys; multi-device key material may conflict with "sole control" requirements (feeds Phase 2)
- Credential recovery, cross-device flows (hybrid/CaBLE), enterprise attestation
- Platform constraints: Apple, Google, Microsoft passkey implementations; what RPs can and cannot observe

### Stream 2: Digital Identity

- W3C Verifiable Credentials Data Model 2.0; DID Core; issuer/holder/verifier triangle
- OpenID4VCI / OpenID4VP; SD-JWT VC; ISO/IEC 18013-5 (mDL) and 23220
- Wallet architectures: EUDI Wallet ARF, Apple/Google Wallet ID credentials, Entra Verified ID
- Trust registries and issuer accreditation models
- Selective disclosure: proving "I am John Smith, authorized to sign" with minimal disclosure
- **Crux:** how is a passkey (authentication key) bound to a verified identity (credential)? Holder binding mechanisms; can a WebAuthn credential serve as a VC holder-binding key?

### Stream 3: Digital Signatures

- PDF signature model: ISO 32000-2, ByteRange, incremental updates, DSS/VRI
- CMS (RFC 5652) / PKCS#7, CAdES / PAdES / XAdES / **JAdES** (ETSI EN 319 122/142/132, TS 119 182 — JAdES matters because it's JSON/JWS-based, closest to WebAuthn's world)
- X.509 chains, CA/trust-list models (EU Trusted Lists), RFC 3161 timestamping, LTV/archival (ETSI EN 319 102 validation)
- **Crux:** can a raw WebAuthn assertion signature be packaged as a valid CMS/PAdES `SignerInfo`, given that WebAuthn signs authenticatorData rather than the CMS SignedAttributes digest? If not, what adaptation layer is needed?
- Remote signing alternative path: Cloud Signature Consortium (CSC) API, ETSI TS 119 432 — where passkeys authorize a server-held signing key (this is how EUDI Wallet QES largely works; PassSign must be positioned relative to it)

### Method & sources

Primary specifications first (W3C, IETF RFCs, ETSI EN/TS, ISO), then implementation documentation (Chromium, WebKit, Android Credential Manager), then WG discussion archives for in-flight extension proposals. Fan-out via deep-research subagents per stream; claims about "what is signed" verified against spec text, not secondary blog posts.

### Deliverable — Document 2: Technology Survey (~20–25 pp)

WebAuthn architecture + flow diagrams · cryptographic primitives inventory · identity/credential architecture · signature container anatomy ("what's inside a signed PDF") · limitations register · candidate integration points for document signing. Ends with hypothesis scorecard for Q1 (technically possible) and preliminary input to Q3 (interoperable).

**Gate 1 exit question:** Which of the three technical paths is viable — (a) direct WebAuthn assertion over document hash, (b) WebAuthn extension for arbitrary signing, (c) passkey-authorized remote/local signing key? This choice shapes everything downstream.

---

## Phase 2 — Legal & Regulatory (Stream 4)

**Goal:** Determine which legal tier a passkey-native signature can reach, in which jurisdictions, under what conditions.

### Scope

- **Primary:** eIDAS — Regulation 910/2014 as amended by 2024/1183 (eIDAS 2.0). SES / AdES / QES definitions; Annex I–III requirements; QSCD certification; "sole control" doctrine; EUDI Wallet as QES enabler; ETSI EN 319 401/411 policy requirements
- **Secondary (comparative):** US ESIGN + UETA (intent, attribution, record integrity — technology-neutral); UK eIDAS + Electronic Communications Act 2000; Singapore ETA 2010 (secure electronic signature tier); Australia ETA 1999
- Case law / regulatory guidance on what evidence satisfies attribution and intent

### Key questions

- Does a synced passkey satisfy AdES requirement "created using data the signatory can, with high confidence, use under their sole control"? What about device-bound passkeys with attestation?
- Can any passkey-native construction reach QES without a QSCD-certified component, or is the realistic ceiling AdES + qualified timestamp?
- Where does the market actually require which tier? (Feeds Phase 3 differentiation.)
- How does the EUDI Wallet's built-in QES capability change the opportunity space?

### Deliverable — Document 3: Legal & Regulatory Landscape (~12–15 pp)

Tier definitions matrix · jurisdiction comparison table · mapping of candidate PassSign constructions (from Gate 1) onto legal tiers · risk register of legal blockers. Scorecard for Q2 (legally viable).

*Note: this document is regulatory research to inform protocol design, not legal advice; formal counsel review is a stated assumption before any RFC publication.*

**Gate 2 exit question:** What is the highest legal tier reachable per technical path, and is that tier commercially sufficient?

---

## Phase 3 — Market & Standards Landscape (Streams 5–6)

**Goal:** Establish differentiation and identify the standardization venue — both determine whether the RFC is worth writing and what shape it takes.

### Stream 5: Existing market

Products: DocuSign, Adobe Acrobat Sign, Dropbox Sign, Yousign, Signicat · identity/wallet: Apple Wallet ID, Google Wallet ID, Entra Verified ID, Login.gov, EUDI Wallet · IAM: Auth0, Okta.

For each: what problem it solves, signature tier achieved, identity-verification model, pricing/lock-in model, developer integration surface, documented failure modes (UX friction, cost, vendor lock-in, weak signer identity binding). Output: capability matrix + failure-mode inventory → the opportunity map.

### Stream 6: Standards bodies

- Who owns what: FIDO Alliance (CTAP, certifications), W3C (WebAuthn, VC), IETF (CMS, JOSE/COSE, RFC track), ETSI (AdES formats, eIDAS technical standards), ISO (PDF, mDL), OpenID Foundation (OID4VC), Cloud Signature Consortium (CSC API)
- Precedents for cross-body compositions (OIDC over OAuth; FIDO2 = W3C + FIDO split)
- Venue analysis: W3C proposal vs. IETF Internet-Draft vs. FIDO extension vs. ETSI profile vs. OpenID Foundation WG — evaluated on IP policy, speed, audience, and where PassSign's novel layer actually sits
- In-flight work that could collide or carry PassSign (WebAuthn WG signing discussions, EUDI ARF, CSC roadmap)

### Deliverable — Document 1: Current State Analysis (~15–20 pp)

Market capability matrix · failure-mode inventory · opportunity map · standards ownership map · recommended standardization venue with rationale. Scorecard for Q4 (differentiated).

**Gate 3 exit question:** Is there a defensible gap that neither incumbents nor in-flight standards will close, and which venue should host the RFC?

---

## Phase 4 — Gap Analysis (Synthesis) ← the pivotal document

**Goal:** Render the verdict: new protocol vs. novel composition — and enumerate exactly what must be specified.

### Method

Every chapter follows your mandated structure: **Current capability → Limitation → Opportunity.** Chapters cover: signing payload semantics, signing intent/ceremony, identity binding, signature container compatibility, trust bootstrapping, revocation, timestamping, long-term validation, recovery, legal tier mapping, developer API surface, user ceremony UX.

Each opportunity is classified:

- **Compose:** existing standard used as-is
- **Profile:** existing standard, constrained/parameterized (spec work, low risk)
- **Extend:** existing standard needs a defined extension (spec + WG engagement)
- **Invent:** nothing exists (highest risk — if any load-bearing item lands here, the hypothesis weakens)

### Deliverable — Document 4: Gap Analysis (~15 pp)

Gap register with classifications · explicit verdict on the composition question · go/no-go recommendation on proceeding to protocol design · scorecard for Q5 (what's missing) and final consolidated scorecard for Q1–Q5.

**Gate 4 exit question:** Does the evidence support proceeding to protocol design, and on which technical path?

---

## Phase 5 — Protocol Design (Stream 7)

**Goal:** Compose the architecture. Only begins after Gate 4 approval.

### Contents

- Actors and roles (signer, RP/document originator, identity issuer, verifier, timestamp authority, trust registry)
- Message flows: signing ceremony, verification ceremony, credential issuance/binding
- Who signs what: exact byte structures, referencing standards chosen in Phase 4
- Trust model and bootstrapping · revocation · timestamping · recovery
- Threat model (STRIDE-style: phishing, key sync compromise, malicious RP, replay, repudiation)
- UX and developer-ergonomics requirements as normative-adjacent design constraints
- Explicit non-goals (guided by research principles: reuse before inventing; open by default; complement eIDAS/ESIGN, don't replace)

### Deliverable — Document 5: Protocol Proposal (~15 pp)

High-level architecture with diagrams · ceremonies · trust model · threat model · open issues list feeding the RFC.

**Gate 5 exit question:** Is the architecture sound and complete enough to formalize?

---

## Phase 6 — RFC Draft v0.1

**Goal:** Formalize Document 5 into RFC conventions matching the venue chosen in Phase 3 (IETF I-D format via kramdown-rfc/xml2rfc, or W3C spec format via ReSpec — decided at Gate 3).

### Deliverable — Document 6: PassSign RFC Draft v0.1

Terminology (RFC 2119/8174 keywords) · protocol messages · normative references · security considerations · IANA/registry considerations if applicable · privacy considerations.

---

## 3. Cross-cutting research principles (adopted from your program)

Reuse before inventing · Open by default · UX is a first-class requirement · Developer ergonomics matter · Legal interoperability over legal disruption. Additionally: **primary sources only for normative claims** — every technical assertion in Docs 1–4 cites the spec section or regulation article it derives from.

## 4. Risks to the program itself

| Risk | Impact | Mitigation |
|------|--------|-----------|
| WebAuthn cannot sign document-bound payloads at all | Kills path (a)/(b) | Path (c) — passkey-authorized signing key — is investigated in parallel in Phase 1, not as a fallback afterthought |
| EUDI Wallet QES already delivers the vision in the EU | Differentiation collapses in primary jurisdiction | Stream 5/6 explicitly scopes EUDI; PassSign may become a profile/bridge rather than a competitor — that's a finding, not a failure |
| Synced passkeys fail "sole control" | Legal ceiling drops to SES/weak AdES | Device-bound + enterprise-attestation variants analyzed as a distinct profile |
| Standards-body politics | Venue rejects or duplicates work | Gate 3 venue analysis includes engagement strategy and prior-art scan of WG mailing lists |

## 5. Logistics

- **Execution:** each phase runs as fanned-out deep-research tasks against primary sources; findings synthesized into the phase document; gate review with you before the next phase starts
- **Outputs:** Markdown files, numbered `passsign-doc{1..6}-*.md`, plus a running `passsign-scorecard.md` tracking Q1–Q5 evidence across phases
- **Order note:** documents are produced in dependency order (Doc 2 → Doc 3 → Doc 1 → Doc 4 → Doc 5 → Doc 6), which differs from their numbering
