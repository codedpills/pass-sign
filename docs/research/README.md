# PassSign Research Program

Research phase outputs for the PassSign RFC ("Passkey-Native Standard for Trusted Electronic Signatures").

**Status:** All six phases complete (2026-07-17). RFC Draft v0.1 published; standards engagement (w3c/webauthn#2078) underway.

## Structure

| Folder | Contents |
|---|---|
| [`plan/`](plan/) | The research program execution plan (phases, streams, gates, decision framework) |
| [`streams/`](streams/) | Raw research documents for the six workstreams, primary-source cited |
| [`deliverables/`](deliverables/) | Phase deliverables: Documents 1–3 (each = executive synthesis + stream research) and the consolidated Q1–Q5 scorecard |

## Reading order

1. [`deliverables/scorecard.md`](deliverables/scorecard.md) — hypothesis verdict, gap classification, gate decisions
2. [`deliverables/doc-4-gap-analysis.md`](deliverables/doc-4-gap-analysis.md) — the pivotal document: 13-chapter gap register, Compose/Profile/Extend/Invent verdict, go/no-go
3. [`deliverables/doc-5-protocol-proposal.md`](deliverables/doc-5-protocol-proposal.md) — the protocol: actors, ceremonies, evidence record, tiers, threat model
4. [`deliverables/doc-6-rfc-draft-v0.1.md`](deliverables/doc-6-rfc-draft-v0.1.md) — formal spec: terminology, normative object/ceremony definitions, security & privacy considerations
2. [`deliverables/doc-2-technology-survey.md`](deliverables/doc-2-technology-survey.md) — Q1/Q3: technical feasibility (Streams 1–3)
3. [`deliverables/doc-3-legal-landscape.md`](deliverables/doc-3-legal-landscape.md) — Q2: legal viability (Stream 4)
4. [`deliverables/doc-1-current-state-analysis.md`](deliverables/doc-1-current-state-analysis.md) — Q4: differentiation and venue (Streams 5–6)

## Headline finding

PassSign is a **novel composition of existing standards, not a new protocol**. Passkeys as deployed cannot sign documents directly (WebAuthn signs only `authenticatorData || SHA-256(clientDataJSON)`), but a passkey ceremony can authorize and evidence standards-conformant signatures today (CSC API / ETSI TS 119 432 remote-signing path), with a native path arriving via the W3C WebAuthn `sign` extension (~2028+). No component requires new cryptography.

## Next steps

- Reference implementation and conformance test-vector suite
- Formal JSON Schema + JAdES mapping annex (Doc 6 §13 open items)
- W3C Community Group formation; continued engagement on w3c/webauthn#2078

*Legal/regulatory content in these documents is design research, not legal advice.*
