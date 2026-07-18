# PassSign

**A passkey-native standard for trusted electronic signatures.**

Billions of people now hold hardware-backed cryptographic keys secured by biometrics — and use them only to log in. Meanwhile, electronic document signing remains fragmented, email-bound, and platform-locked. PassSign is an open protocol that extends the passkey ceremony into legally meaningful, cryptographically verifiable document signing: sign a contract the way you sign into a website.

## The core idea

Passkeys as deployed **cannot sign documents** — a WebAuthn assertion only ever says "I authenticated." PassSign's research phase established that this is a composition problem, not a technology gap: every required component already exists. The protocol therefore makes the passkey the **authorization and evidence root** of every signature:

1. The application counter-signs a **manifest** committing it to exactly what the signer was shown.
2. The signer's passkey approves an assertion cryptographically bound to that manifest (one biometric gesture).
3. A **Signing Service** verifies that approval and produces a fully standard signature (PAdES — opens with a valid signature in existing PDF tooling).
4. Everything is bundled into an **evidence record** — a plain JWS any third party can verify **offline, forever**, with a copy held by the signer. No verifiable evidence record → not a PassSign signature.

Three assurance tiers (Baseline / Assured / Qualified) make claims machine-checkable, mapping to SES → AdES → QES under eIDAS and to strong attribution evidence under ESIGN/UETA — complementing existing legal frameworks, never replacing them.

## Why it's different

| Incumbent reality | PassSign |
|---|---|
| Signer identity = access to an email inbox; real verification costs ~$2.40/attempt | Identity verified once via wallet credential (EUDI, mDL…), reused at zero marginal cost |
| Audit trail = platform-sealed log; trust the vendor, hope it still exists | Evidence record verifies offline against public keys — independent of any platform's survival or honesty |
| Emailed signing links = phishing surface | Origin-bound passkey ceremony; no account creation, no capability URLs |
| Proprietary APIs, per-envelope pricing, QES behind sales calls | Open spec: one manifest API + one webhook; verification implementable from the spec alone |

## Status

**Research and protocol design complete (Phases 1–5 of the program); RFC draft v0.1 is next.** Six research streams — WebAuthn internals, digital identity (VC/OID4VP/wallets), signature standards (CMS/AdES), legal frameworks (eIDAS, ESIGN/UETA, UK/SG/AU), market analysis, and standards-body landscape — all primary-source cited. Verdict: **PassSign is a novel composition of existing standards**; of thirteen analyzed gaps, the only "invent" item is the signing ceremony itself — which is this spec.

| Milestone | Status |
|---|---|
| Concept draft | ✅ [`docs/concept-draft.md`](docs/concept-draft.md) |
| Research program (6 streams, 3 phase documents, scorecard) | ✅ [`docs/research/`](docs/research/) |
| Gap analysis & verdict | ✅ [`docs/research/deliverables/doc-4-gap-analysis.md`](docs/research/deliverables/doc-4-gap-analysis.md) |
| Protocol proposal | ✅ [`docs/research/deliverables/doc-5-protocol-proposal.md`](docs/research/deliverables/doc-5-protocol-proposal.md) |
| RFC draft v0.1 | 🔜 next |
| Reference implementation & test vectors | planned |
| W3C Community Group + standards engagement | planned |

## Repository layout

```
docs/
├── concept-draft.md        # original concept (v0.1)
└── research/               # research program (start at research/README.md)
    ├── plan/               # execution plan and phase gates
    ├── streams/            # six primary-source research streams
    └── deliverables/       # phase documents 1–5 + consolidated scorecard
```

New here? Read [`docs/research/deliverables/scorecard.md`](docs/research/deliverables/scorecard.md) for the verdict in two pages, then the protocol proposal.

## Design principles

Reuse before inventing · Open by default (no single-operator dependency — signing services are substitutable by design) · User experience is a first-class requirement · Developer ergonomics matter · Legal interoperability over legal disruption.

## Contributing

The project is pre-RFC; the most valuable contributions right now are review of the protocol proposal's threat model and open issues (§9/§11), prior-art pointers we missed, and legal-framework review for additional jurisdictions. Open an issue to discuss.

*Legal and regulatory statements in this repository are design research, not legal advice.*
