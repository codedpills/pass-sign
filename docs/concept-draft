PassSign: A Passkey-Native Standard for Trusted Electronic Signatures

Version: 0.1 (Concept Draft)

⸻

Executive Summary

The widespread adoption of passkeys has fundamentally changed how users authenticate online. For the first time, billions of people possess hardware-backed cryptographic key pairs secured by biometrics, yet this technology is almost exclusively used for authentication.

At the same time, electronic document signing remains fragmented, cumbersome, and largely dependent on proprietary platforms, uploaded images of handwritten signatures, or cryptographic certificates that are too complex for the average user.

This document proposes PassSign, an open protocol and ecosystem that extends the simplicity and security of passkeys into the world of electronic document signing.

Rather than replacing existing standards such as PDF digital signatures or eIDAS, PassSign aims to provide a modern, developer-friendly, interoperable layer that enables any application to request legally meaningful, cryptographically verifiable signatures with the same ease as signing into a website using Face ID or Windows Hello.

⸻

The Problem

The Current State of Electronic Signatures

Electronic signatures today generally fall into four categories.

1. Visual Signatures

Examples include:

* Images of handwritten signatures
* Mouse-drawn signatures
* Typed names
* Initials

These signatures primarily serve as visual representations of intent.

Problems:

* Easily copied
* Easily forged
* No cryptographic proof
* No identity verification
* No proof that the signer actually approved the document

⸻

2. Platform-Based Electronic Signatures

Services such as DocuSign and Adobe Acrobat Sign create trust by maintaining an audit trail.

Typical evidence includes:

* Email address
* IP address
* Device information
* Timestamp
* SMS verification
* Audit logs

These systems work well but introduce platform dependency.

Trust exists because the service provider maintains evidence—not because the document itself contains independently verifiable proof.

⸻

3. PKI-Based Digital Signatures

Cryptographic signatures attached directly to PDF documents provide:

* Integrity
* Authenticity
* Non-repudiation

However, they suffer from poor usability.

Users often require:

* Certificates
* Smart cards
* Enterprise software
* Manual certificate installation

As a result, adoption remains largely limited to governments and large organizations.

⸻

4. Qualified Electronic Signatures

Regulatory frameworks such as eIDAS provide legally equivalent signatures to handwritten ones.

These systems offer the highest assurance but often involve:

* Identity verification
* Government-issued certificates
* Qualified trust service providers

The security is excellent.

The user experience is not.

⸻

The Core Observation

Authentication has become dramatically easier.

Signing has not.

Today, logging into a bank account can be accomplished in seconds using Face ID.

Signing a document often still involves:

* Drawing a signature
* Uploading a PNG
* Typing a name
* Receiving email verification
* Downloading a PDF

The technology gap is striking.

Passkeys demonstrate that modern cryptography can become invisible to end users.

Document signing should benefit from the same transformation.

⸻

Vision

Signing Should Feel Like Authentication

A user should never have to think about certificates, keys, or cryptography.

The ideal signing flow becomes:

1. Click “Sign Document”
2. Review the document
3. Confirm with Face ID, Touch ID, Windows Hello, or Android Biometrics
4. Document is cryptographically signed
5. Anyone can independently verify the signature

The entire experience should take seconds.

⸻

Core Principles

PassSign is founded upon the following principles.

Security by Default

Private keys never leave the user’s device.

⸻

Open Standard

No vendor lock-in.

Any software can implement the protocol.

⸻

Identity Agnostic

Identity should not be tied to one provider.

Possible issuers include:

* Governments
* Banks
* Universities
* Employers
* Licensed identity providers
* Trust service providers

⸻

Privacy First

Verifiers should learn only the information necessary to validate the signature.

Users should retain control over which identity attributes are disclosed.

⸻

Long-Term Verifiability

Documents should remain independently verifiable years after they are signed.

Verification should not depend on the continued existence of a single commercial platform.

⸻

System Overview

The ecosystem consists of four primary actors.

Identity Provider

Responsible for verifying real-world identity.

Examples:

* Government
* Bank
* University
* Employer
* Qualified Trust Service Provider

Outputs:

A digitally signed identity credential.

⸻

User Wallet

Stores:

* Passkey
* Signing credentials
* Identity credentials

The wallet may be integrated into:

* Apple Wallet
* Google Wallet
* Browser
* Password manager
* Dedicated signing wallet

⸻

Signing Application

Any application capable of requesting signatures.

Examples:

* HR software
* Contract platforms
* Banking applications
* Healthcare systems
* PDF readers
* Government portals

⸻

Verification Service

Responsible for verifying:

* Signature validity
* Document integrity
* Credential authenticity
* Revocation status
* Timestamp
* Identity assurance level

Verification should be possible by anyone.

⸻

High-Level Signing Flow

Step 1

Application prepares document hash.

↓

Step 2

Application requests signature.

↓

Step 3

Browser or operating system prompts:

“Do you want to sign this document?”

↓

Step 4

User authenticates with biometrics.

↓

Step 5

Passkey signs document hash.

↓

Step 6

Signature package generated.

↓

Step 7

Document contains:

* Cryptographic signature
* Timestamp
* Credential reference
* Identity assurance metadata

↓

Step 8

Recipient independently verifies signature.

⸻

Identity Assurance Levels

Not every signature requires the same level of trust.

PassSign should support multiple assurance levels.

Level 0

Anonymous cryptographic signature.

Proves:

The same device signed.

⸻

Level 1

Email verified.

⸻

Level 2

Phone verified.

⸻

Level 3

Bank verified.

KYC completed.

⸻

Level 4

Employer verified.

Useful for internal approvals.

⸻

Level 5

Government verified.

Equivalent to national digital identity.

⸻

Level 6

Qualified Electronic Signature.

Compatible with existing legal frameworks where applicable.

⸻

Proposed Technical Building Blocks

Rather than inventing new cryptography, PassSign should compose existing standards.

Potential components include:

* WebAuthn
* Passkeys
* FIDO2 authenticators
* W3C Verifiable Credentials
* Decentralized Identifiers (optional)
* COSE signatures
* JWS/JWT where appropriate
* RFC 3161 timestamping
* Existing PDF digital signature standards
* Certificate Transparency-inspired public logs (optional)

The innovation lies in orchestration rather than cryptography.

⸻

Potential Browser API

One long-term possibility is a standardized browser API.

Example:

navigator.signDocument()

Similar to:

navigator.credentials.get()

The browser would display:

⸻

Sign Document

Employment Agreement.pdf

SHA-256

Identity:
Government Verified

Sign?

[Cancel] [Face ID]

⸻

⸻

Signature Package

Rather than embedding only a visual signature, each signed document could include:

* Document hash
* Public key identifier
* Signature
* Timestamp
* Credential chain
* Assurance level
* Revocation information
* Optional verification URL

⸻

Verification

Verification should answer several questions.

Integrity

Has the document changed?

⸻

Authenticity

Was it signed using the expected cryptographic key?

⸻

Identity

Who issued the identity credential?

⸻

Trust

How strong was the identity verification?

⸻

Revocation

Is the signing credential still valid?

⸻

Time

When was the signature created?

⸻

Developer Experience

One of PassSign’s greatest strengths should be its simplicity.

Possible SDK:

const signature = await PassSign.sign(document)

Verification:

const result = await PassSign.verify(document)

Result:

{
  "valid": true,
  "identity": "Government Verified",
  "timestamp": "...",
  "issuer": "...",
  "revoked": false
}

The complexity should remain inside the protocol.

⸻

Benefits

Users

* No passwords
* No uploaded signatures
* Face ID approval
* Secure by default
* Device-bound keys

⸻

Businesses

* Lower fraud
* Easier onboarding
* Reduced support
* Standard API
* Vendor independence

⸻

Governments

* Easier adoption of digital identity
* Better interoperability
* Increased trust

⸻

Developers

* Single signing protocol
* Modern SDKs
* Browser support
* Platform independence

⸻

Challenges

Identity Binding

A passkey proves possession of a key.

It does not prove legal identity.

Identity providers remain essential.

⸻

Legal Recognition

Different jurisdictions define electronic signatures differently.

PassSign should complement—not replace—existing regulations such as eIDAS, ESIGN, and UETA.

⸻

Revocation

Devices can be lost.

Credentials may expire.

Revocation infrastructure is required.

⸻

Long-Term Cryptography

Documents may need to remain valid for decades.

Algorithm agility must be built into the protocol.

⸻

Cross-Platform Support

Support is required across:

* Browsers
* Mobile devices
* Desktop operating systems
* PDF viewers
* Enterprise systems

⸻

Privacy

Identity disclosure should be selective.

Many signatures require only proof of authorization—not full identity.

Zero-knowledge techniques or selective disclosure credentials could eventually enhance privacy.

⸻

Possible Roadmap

Phase 1

Research

* Study WebAuthn
* Study Verifiable Credentials
* Study PDF digital signatures
* Study eIDAS
* Define protocol

⸻

Phase 2

Developer SDK

* JavaScript SDK
* Verification SDK
* Reference server

⸻

Phase 3

Reference Wallet

Desktop and mobile wallet capable of storing signing credentials.

⸻

Phase 4

Browser Extension

Prototype “Sign with PassSign” experience.

⸻

Phase 5

Reference Identity Provider

Simple demo issuer for development purposes.

⸻

Phase 6

Open Standard

Publish protocol.

Invite community participation.

Establish governance.

⸻

Long-Term Vision

PassSign is not intended to become another document-signing platform.

Instead, it aspires to become the infrastructure layer upon which thousands of document-signing applications are built.

Just as:

* OAuth standardized delegated authorization,
* OpenID Connect standardized identity,
* WebAuthn standardized passwordless authentication,

PassSign could standardize trusted, passkey-native electronic signatures.

The ultimate goal is simple:

Signing a document should become as secure, effortless, and universally supported as logging into a website with Face ID.
