# Stream 3: Digital Signature Standards — How Documents Are Signed Today

**PassSign research program — Stream 3 of 6**
**Date:** 2026-07-17
**Scope:** PDF/CMS/AdES signature anatomy; feasibility analysis of packaging WebAuthn (passkey) assertions into standard signature containers; prior art; X.509 issuance for passkeys; timestamping/LTV; verification ecosystem; remote-signing standards (CSC/ETSI/CEN).
**Method note:** Normative claims are cited to primary specifications where possible. Claims that could not be verified against a fetched primary source in this run are marked **[unverified]**. This document builds on a confirmed prior finding: GitHub searches found **no dedicated WebAuthn+CMS/AdES integration projects**, and the community-curated awesome-webauthn list contains no document-signing/AdES projects [19].

---

## 1. PDF Signature Anatomy (ISO 32000-2 §12.8)

A digitally signed PDF embeds the signature inside the document itself, using four interlocking mechanisms:

**Signature dictionary and ByteRange.** A signature field's value is a signature dictionary containing (among others) `/Filter` (handler, conventionally `Adobe.PPKLite`), `/SubFilter` (encoding of the signature blob), `/Contents` (the signature blob itself as a hexadecimal string), and `/ByteRange` — an array of four integers `[offset1 length1 offset2 length2]` defining two spans of the file's exact bytes that are covered by the signature. The two spans are everything **except** the `/Contents` hex string itself: the signature cannot cover its own value, so a "hole" is left for it. The signing software computes the digest over the ByteRange bytes, produces the signature container, and writes it into the pre-allocated (zero-padded) `/Contents` gap. Consequence: the signed object is the *serialized file bytes*, and any byte outside the hole is tamper-evident. (ISO 32000-2 §12.8.1; ISO 32000-2 is paywalled — anatomy corroborated by PDF industry documentation [2][3].) **[unverified: exact ISO clause wording; the structure itself is uncontroversial and implemented identically by Adobe, Apryse, Nutrient, iText, open-source pdfbox, etc.]**

**SubFilter values.** Legacy Adobe signatures use `/adbe.pkcs7.detached` (detached CMS over ByteRange bytes) or `/adbe.pkcs7.sha1`. PAdES signatures use `/ETSI.CAdES.detached` (ETSI EN 319 142-1), and document timestamps use `/ETSI.RFC3161` [3][6]. The important precedent: **new SubFilter values have been added to the PDF ecosystem before** (ETSI added two), so the container has an extension point — but every new SubFilter required standardization plus multi-year validator rollout.

**Incremental updates.** PDF supports appending revisions after the `%%EOF` without touching earlier bytes. A second signature, a document timestamp, or LTV data is added as an incremental update; each signature's ByteRange covers the file *up to its own revision*. This is how serial multi-party signing and post-signature augmentation (LTV, archival timestamps) work without invalidating earlier signatures.

**DocMDP / certification signatures.** The first ("certification") signature may include a `/Reference` array with a DocMDP transform whose `/P` parameter constrains permitted post-signing changes: 1 = no changes, 2 = form fill-in and signing, 3 = plus annotations. Approval signatures (ordinary signatures) do not carry DocMDP. Validators check that incremental updates after a certified revision stay within the declared permission level.

**What exactly is embedded:** the DER-encoded CMS `SignedData` structure sits inside `/Contents` — signer certificate(s) and chain, signed and unsigned attributes, the signature value, optionally an embedded RFC 3161 timestamp token, and (for LTV) revocation material either inside the CMS or in the document-level `/DSS` dictionary (Section 6). A signed PDF is therefore a *self-contained verification package*: document bytes + CMS + certificates + timestamps + revocation data.

**Design implication for PassSign:** the PDF layer is agnostic about what is inside `/Contents`; the byte-coverage model only requires that the signature scheme ultimately bind to a digest of the ByteRange bytes. The constraint lives one layer down, in CMS.

---

## 2. CMS / PKCS#7 SignerInfo: The Byte Model (RFC 5652)

CMS `SignedData` (RFC 5652 §5) contains `encapContentInfo` (absent/detached for PDF), a certificate bag, and one or more `SignerInfo` structures (§5.3) with: `sid` (issuer+serial or subjectKeyIdentifier identifying the signer's **X.509 certificate**), `digestAlgorithm`, optional `signedAttrs`, `signatureAlgorithm`, `signature`, optional `unsignedAttrs` [1].

**The critical byte-level fact:** when `signedAttrs` is present (mandatory in all AdES profiles), **the signature is NOT computed over the document**. Per RFC 5652 §5.4, the message-digest calculation is performed over "the complete DER encoding of the SignedAttrs value," re-tagged with an EXPLICIT `SET OF` tag instead of the `IMPLICIT [0]` tag used on the wire; the signature (§5.5) is then computed over that DER structure [1]. The document is bound only *indirectly*: the `signedAttrs` set MUST contain (§11):

- `content-type` (§11.1) — OID of the encapsulated content type;
- `message-digest` (§11.2) — the hash of the actual content (for PDF: the ByteRange bytes);
- optionally `signing-time` (§11.3) — though PAdES baseline uses the signature dictionary's `/M` entry instead and prohibits the CMS signing-time attribute **[unverified: exact PAdES clause]**;
- in AdES: `signing-certificate-v2` (ESS, RFC 5035), binding the signer's certificate hash into the signed bytes.

So the chain of trust in every standards-compliant signed document today is:

```
document bytes → SHA-2 digest → messageDigest attribute
             → DER(SignedAttributes) → SHA-2 digest → raw signature primitive (ECDSA/RSA/EdDSA)
```

**This two-stage indirection is the exact structural analogue of WebAuthn's own indirection** (challenge → clientDataJSON → hash → concatenated with authenticatorData → signature) — but the two indirection formats are byte-incompatible, which is the crux analyzed in Section 4.

---

## 3. The AdES Family: CAdES, XAdES, PAdES, JAdES

ETSI's Advanced Electronic Signature (AdES) formats are profiles layered on generic container formats, adding the attributes needed for legal-grade, long-lived signatures. All four share a common level model — **B-B** (basic: signed attributes incl. signing certificate reference), **B-T** (adds a trusted RFC 3161 timestamp on the signature), **B-LT** (adds all validation material: certificate chain + revocation data), **B-LTA** (adds periodic archival timestamps for decades-scale validity) — and a common validation procedure defined in ETSI EN 319 102-1 [8].

- **CAdES** (ETSI EN 319 122-1) profiles CMS: mandatory `signing-certificate-v2`, `signature-time-stamp` unsigned attribute for -T, archive timestamps for -LTA. Applicable to any file type [5].
- **XAdES** (EN 319 132-1) is the XML-DSig equivalent, with `QualifyingProperties`. Signing input is the canonicalized (C14N) `SignedInfo` element — XML canonicalization adds complexity irrelevant to PassSign except as a cautionary tale.
- **PAdES** (EN 319 142-1) profiles CAdES for embedding in PDF via `/ETSI.CAdES.detached`, adds PDF-specific rules (one SignerInfo only, ByteRange must cover the whole revision, `/M` for time, DSS/VRI for LT, document timestamps for LTA) [6].
- **JAdES** (ETSI TS 119 182-1) profiles **JWS (RFC 7515)**: the newest and least deployed member, targeted at JSON/API ecosystems [7].

### JAdES focus: how close is its signing input to a WebAuthn assertion?

JWS computes the signature over the **JWS Signing Input**: `ASCII(BASE64URL(UTF8(protected header)) || '.' || BASE64URL(payload))` (RFC 7515 §5.1) [4]. JAdES adds ETSI header parameters — `sigT` (claimed signing time), `x5c`/`x5t#S256` (certificate binding, the JWS-native equivalent of signing-certificate-v2), `sigD` (a detached-payload mechanism referencing external documents by URI + digest, so the effective payload can be a compact digest manifest rather than the document itself), and the unprotected `etsiU` array carrying timestamps and validation data for T/LT/LTA levels [7].

**Answer to the research question: structurally close, byte-wise incompatible.** A WebAuthn assertion signature is computed over `authenticatorData || SHA-256(clientDataJSON)` (WebAuthn L3 §6.3.3; verified in §7.2: "verify that sig is a valid signature over the binary concatenation of authData and hash") [9]. The JWS Signing Input is a string over the base64url alphabet plus `.`; `authenticatorData` is arbitrary binary (rpIdHash ‖ flags ‖ signCount ‖ extensions). The two byte strings cannot be made to coincide except by contrivance, so **no existing JWS `alg` value can validate a WebAuthn assertion as a JAdES signature**. However, JAdES is the *most amenable* container for a new profile, because:

1. JOSE has a lightweight, active registry for new `alg` values (vs. new ASN.1 AlgorithmIdentifier OIDs plus DER parameter structures for CMS);
2. `clientDataJSON` is itself JSON — carrying it in a JWS unprotected header is idiomatic, whereas in CMS it needs a new ASN.1 attribute;
3. the `sigD` detached mechanism already establishes the pattern "payload = digest manifest of external documents," which maps naturally onto "challenge = digest of signing input."

A hypothetical `"alg": "WebAuthn-ES256"` could define: challenge := BASE64URL(SHA-256(JWS Signing Input)); signature := assertion signature; unprotected header carries `authenticatorData` + `clientDataJSON`; verifier recomputes both indirection layers. This is exactly the kind of two-layer verification the ecosystem has never standardized. No such JOSE registration or ETSI work item was found **[negative finding; unverified as exhaustive]**.

---

## 4. **The Crux: WebAuthn Assertions Inside Standard Containers — Analysis & Prior Art**

### 4.1 The incompatibility, stated precisely

| | CMS/CAdES/PAdES | JWS/JAdES | WebAuthn assertion |
|---|---|---|---|
| Bytes actually signed | DER(SignedAttributes), EXPLICIT SET OF re-tag (RFC 5652 §5.4) [1] | ASCII(B64URL(header)‖'.'‖B64URL(payload)) (RFC 7515 §5.1) [4] | authenticatorData ‖ SHA-256(clientDataJSON) (WebAuthn §6.3.3) [9] |
| Document binding | messageDigest signed attribute | payload or sigD digest manifest | RP-chosen `challenge` inside clientDataJSON |
| Signer identity | X.509 cert referenced by `sid` + signing-certificate-v2 | `x5c`/`x5t#S256` | bare COSE public key, no certificate |
| Extra signed context | attributes chosen by signer | protected header | origin, type, crossOrigin (clientDataJSON); rpIdHash, UV/UP flags, signCount (authData) |

The underlying primitives are compatible — ES256/EdDSA/RS256 are all valid CMS and JWS algorithms — but an authenticator **refuses to sign caller-chosen bytes**; it always injects its own envelope. This is a deliberate security property (domain separation prevents an RP from tricking an authenticator into signing, e.g., a Bitcoin transaction or another RP's login challenge), and it is precisely what breaks container compatibility.

### 4.2 Could a CMS/AdES profile treat the assertion as SignatureValue?

Technically yes — cryptographic verifiability survives. The profile would be:

1. Build normal PAdES `SignedAttributes` (content-type, messageDigest over ByteRange, signing-certificate-v2).
2. Compute `H = SHA-256(DER-EXPLICIT(SignedAttributes))` and set `challenge = BASE64URL(H)` in the WebAuthn `get()` call.
3. Place the assertion's raw signature bytes in `SignerInfo.signature`.
4. Carry `authenticatorData` and `clientDataJSON` in new attributes (unsigned attributes suffice — they are integrity-protected transitively, because tampering with either changes the bytes the signature verifies over).
5. Define a new `signatureAlgorithm` OID, e.g. `webauthn-assertion-with-ES256`, whose verification procedure is: reconstruct `authData ‖ SHA-256(clientDataJSON)`, verify ECDSA; parse clientDataJSON; check `type == "webauthn.get"` and `challenge == BASE64URL(SHA-256(DER(SignedAttributes)))`; then proceed with standard CMS/AdES validation.

**What breaks in existing validators — everything at the last mile:**

- **Declared-algorithm mismatch.** If the profile lies and declares `ecdsa-with-SHA256`, every validator (Adobe, EU DSS, OpenSSL) computes the signature check over DER(SignedAttributes) and gets **TOTAL-FAILED** — a *red X*, worse than unknown. If it declares a new OID, validators return unknown-algorithm errors (EU DSS: INDETERMINATE; Adobe: "signature validity is unknown") [8][20][21]. There is no graceful degradation path.
- **Certificate requirement.** `sid` must reference an X.509 certificate; AdES-B additionally mandates signing-certificate-v2. A bare passkey has no certificate — Section 5's problem is a hard dependency, not an optional nicety.
- **PAdES SubFilter.** A conforming PAdES validator checks the SubFilter; a WebAuthn-CMS blob would need a new value (precedent: `/ETSI.RFC3161` was added for document timestamps), again requiring standardization + rollout.
- **EN 319 102-1 validation model.** The standardized validation process assumes X.509 chain building to a trust anchor, revocation checking, and signature verification per registered algorithms [8]. A WebAuthn profile must either map onto this model (cert for the passkey + new algorithm entry in ETSI TS 119 312 algorithm catalogue **[unverified: 119 312 process details]**) or define a parallel validation standard.

**Verdict:** an "assertion-as-SignatureValue" profile is *cryptographically sound and specifiable in ~10 pages*, but yields signatures that **zero deployed validators accept today**. Its value is as a standardization target, not an interop hack.

### 4.3 Prior art survey

- **Yubico, "Using WebAuthn for Signing" (developer concept doc)** — the canonical description of the *naïve* approach: replace the random challenge with the document hash, archive clientDataJSON + authenticatorData + signature, verify against the registration public key [11]. Explicitly notes the signature is "for an object comprising both the client data … and authenticator data," i.e., a **proprietary verification format**. No container, no certificate, no timestamp, no AdES mapping. This confirms the approach works but stops exactly where PassSign would begin.
- **W3C WebAuthn "sign" extension — PR #2078 (emlun/Yubico, open as of May 2025)** — the decisive piece of prior art [10]. It creates a **separate signing key pair** associated with (but distinct from) the credential key, which signs **"the given input unaltered"** — no authenticatorData/clientDataJSON envelope. Distinctness preserves domain separation while enabling raw signatures. Motivating use cases in the PR: "attested, hardware-bound signing keys for … digital identity wallets" and "general-purpose digital signatures, with seamless interoperability with existing cryptographic protocols." Status signals from the PR thread: Yubico has "rough internal proof-of-concept prototypes … nothing yet ready to share"; no other WG implementation commitments; design iteration moved to yubicolabs/webauthn-sign-extension; a companion `kem` extension is planned; the author engaged the IETF COSE/JOSE lists [10][12]. EUDI-wallet actors (ve7jtb) see it as a candidate WSCD interface for wallets.
  **Key insight: even if #2078 ships, PassSign-style container/profile work is STILL needed.** The extension solves only the byte-format problem (the authenticator can now sign DER(SignedAttributes) or a JWS Signing Input directly, producing a signature standard validators can verify mechanically). It does **not** provide: (a) an X.509 certificate binding the sign-extension public key to a legal identity; (b) an AdES profile saying which attributes/headers to use and how attestation is conveyed; (c) trust-list acceptance; (d) timestamping/LTV integration; (e) any normative statement that a UV-gated hardware signature meets AdES/QES "sole control" requirements. The extension is an enabler, not a solution.
- **Yubico/webauthn-sign-kem-extensions repo** — earlier draft of the same extensions; superseded by PR #2078 [12].
- **FIDO Alliance × eIDAS white papers (2020, 2023)** — FIDO's own position is **not** "sign documents with passkeys" but "use FIDO2 as the *authentication* layer toward a QTSP that holds the signing key server-side" ([13][14]; see Section 8). This is the strongest signal that industry consensus routed around the container problem rather than solving it.
- **Academic:** arXiv 2601.06554 (Jan 2026, Bicakci et al.) is the *inverse* problem — using QES-grade PKCS#11 hardware to back a virtual FIDO2 authenticator — confirming the gap is recognized but attacked from the opposite direction [16]. No paper was found that packages WebAuthn assertions into CAdES/PAdES/JAdES **[negative finding; unverified as exhaustive]**.
- **GitHub / community:** prior-run finding confirmed — no dedicated WebAuthn+AdES projects; awesome-webauthn lists none [19]. The OR13 comment on PR #2078 describes ad-hoc "overloading the challenge to sign arbitrary data … it's all gross" for wallet credential binding [10] — practitioners hack around the gap; nobody has standardized through it.
- **ETSI/W3C formal work items:** no ETSI ESI work item on FIDO/WebAuthn signature formats was found **[unverified as exhaustive; ETSI work programme not directly searchable in this run]**.

**Conclusion of crux:** the composition gap is real, recognized in fragments across four communities (W3C, FIDO, ETSI, academia), and unclaimed as a unified standardization effort.

---

## 5. X.509 / CA Model: Certifying a Passkey Public Key

Every AdES signature requires the signer's X.509 certificate. Three issuance paths for a passkey public key:

**(a) Classic CSR — blocked.** PKCS#10 (RFC 2986) proof-of-possession requires the requester to sign the DER-encoded `CertificationRequestInfo` with the subject private key [17]. A WebAuthn credential *cannot* produce that signature: the authenticator will only emit `Sign(authData ‖ SHA-256(clientDataJSON))`, never a raw signature over caller-supplied DER. Embedding the CSR hash in the challenge produces a signature that is *not* a valid PKCS#10 self-signature, so standard CA software rejects it. (With the #2078 sign extension, raw CSR signing becomes possible — another reason the extension matters.)

**(b) Attestation-based issuance — viable today, requires CA policy work.** At credential creation, the authenticator can return an **attestation statement**: a signature by an authenticator-model key (chaining to the vendor and verifiable against FIDO MDS) over data that includes the newly created credential public key [9]. A CA could accept this as key-binding evidence in lieu of PKCS#10 PoP — the attestation proves the key was generated on, and is bound to, a certified authenticator, which is arguably *stronger* than PoP. Precedent exists: ACME `device-attest-01` (RFC 9720-track draft; used by Apple Managed Device Attestation) issues certificates based on device attestations **[unverified: exact ACME draft/RFC status]**. Caveats: consumer platform passkeys frequently return attestation `none` (Apple in particular suppresses it for synced passkeys) **[unverified for current platform behavior]**, so this path works best with security keys and enterprise-attestation deployments; RFC 4211 (CRMF) already admits PoP methods other than signature, giving standards cover for CA/Browser-Forum-style policy definitions.
**(c) Challenge-based issuance.** The CA runs a WebAuthn ceremony itself (challenge = CA-chosen nonce, optionally binding the CSR content hash), verifies the assertion with the claimed public key, and treats it as PoP. Cryptographically sound; not describable in existing CA protocol standards (EST/CMP/ACME) without a new PoP type — a small, well-scoped standardization item.

Additional model frictions (well summarized from a PKI-practitioner perspective in [18]): passkeys are per-RP by design (privacy), so a certified passkey becomes a cross-context identity — reintroducing exactly the linkability WebAuthn avoids; no revocation infrastructure exists on the passkey side; synced passkeys (private key material replicated via cloud keychains) may fail "sole control" requirements for higher assurance levels — a Stream 4 legal question, flagged here.

---

## 6. Timestamping and Long-Term Validation

- **RFC 3161:** a TSA returns a `TimeStampToken` — itself a CMS SignedData over `TSTInfo` containing the requester's `MessageImprint` (hash) and genTime [15 n/a; RFC 3161]. In CAdES/PAdES-T, the imprint is the hash of the `SignerInfo.signature` value, proving the signature existed at time T. **Note for PassSign: RFC 3161 is format-agnostic — it timestamps a hash. A WebAuthn assertion signature can be timestamped today with zero changes.** This component composes cleanly.
- **LTV / DSS in PDF:** PAdES B-LT stores the full chain, OCSP responses and CRLs in the document-level `/DSS` dictionary (with per-signature `/VRI` entries), added by incremental update, so validation succeeds after certificate expiry using data captured at signing time [6]. B-LTA adds `/DocTimeStamp` (SubFilter `/ETSI.RFC3161`) document timestamps re-applied before algorithm/TSA-cert obsolescence, chaining evidence for decades.
- **PassSign gap:** LTV is built entirely around X.509 revocation artifacts. A passkey-native scheme must define what "revocation data" means (credential still on RP allow-list? attestation chain status? nothing?). If the passkey key is certified (Section 5), standard LTV applies unchanged — another argument for the certificate route.

---

## 7. Verification Ecosystem: What "Validates in Existing Tooling" Requires

- **Adobe Acrobat/Reader** validates against the **AATL** (Adobe Approved Trust List) and EUTL; green-check requires: chain to an AATL/EUTL root, valid revocation status, intact ByteRange digest, and a signature that verifies under a *recognized* AlgorithmIdentifier [21]. Anything else renders "validity unknown" or "invalid" — and in practice, a yellow/red badge is a document that counterparties reject.
- **EU DSS library** (EC Digital Building Blocks, open source) is the reference implementation of EN 319 102-1 validation and the engine behind many national validators; it emits TOTAL-PASSED / INDETERMINATE / TOTAL-FAILED with sub-indications [8][20]. Nonstandard algorithm OIDs or missing certificates yield INDETERMINATE at best.
- **Fate of nonstandard schemes:** DocuSign-style platform "signatures" historically wrap a *platform* certificate (the vendor signs, the user merely clicks), which validates in Adobe because the vendor cert is AATL-listed — the user's own key never signs. Proprietary formats that skip CMS entirely (e.g., naive WebAuthn archives per [11]) are invisible to the entire validation ecosystem and carry evidentiary weight only via the operator's audit logs. History (XML-DSig custom profiles, national one-off formats) shows schemes that don't validate in Adobe/DSS get relegated to closed ecosystems.
- **Bar for PassSign:** either (i) produce byte-perfect PAdES/CAdES via a certified key (server-held or #2078-extension), which validates everywhere immediately, or (ii) standardize a new profile through ETSI + get validator implementations (DSS first — it is open source and the realistic beachhead) — a multi-year path.

---

## 8. Remote Signing: CSC, ETSI TS 119 432, CEN EN 419 241-2 — Where Passkeys Already Fit

The remote-signing stack is today's *only* production-grade junction of passkeys and standard signatures:

**CSC API v2.2 (Cloud Signature Consortium, published 6 Nov 2025)** [22][23]: the signing key lives in a QTSP/RSSP HSM. Flow: OAuth2/service auth → `credentials/list` → `credentials/info` → credential authorization → `signatures/signHash` (base64url hashes in, raw signature values out) → client assembles CAdES/PAdES locally. Credential authorization is either explicit **SAD** (Signature Activation Data, from `credentials/authorize`) or, new in v2.x, an **OAuth2 access token with `credential` scope**, which "can be used instead of a classical SAD" and can be hash-bound so the authorization cannot be replayed to sign different content [23]. Because authorization rides on OAuth2, **a passkey can already be the user-facing authentication/authorization factor** at the RSSP's authorization server today — with zero changes to any signature format. The output is perfect PAdES that Adobe validates.

**ETSI TS 119 432 V1.3.1 (2026-03)** standardizes protocols for remote signature creation (XML and JSON bindings), aligned with the CSC API; the EC has assessed v1.2.1/1.3.1 and an EN is planned for ~2027-06 [24][25]. **CEN EN 419 241-2** is the Common Criteria protection profile for the QSCD server-signing system: a certified **Signature Activation Module (SAM)** inside the HSM boundary enforces **SCAL2** ("sole control assurance level 2") — the SAD must cryptographically bind (signer identity, signing key, hash of the data to be signed), so not even the QTSP's operators can activate the key without the signer's fresh authorization [26] **[unverified: exact EN 419 241-2 clause; corroborated by Entrust SAM service description and Thales/Securosys documentation]**.

**Where WebAuthn appears:** the FIDO Alliance × eIDAS white papers explicitly propose FIDO2 as the high-assurance authentication mechanism toward QTSPs for remote QES [13][14]. The natural deep integration — **derive the SAD from a WebAuthn assertion whose challenge embeds the DTBS/R hash** — gives cryptographic (not merely session-level) sole-control binding: the passkey signature itself attests "this user authorized this exact document hash." Whether any QTSP's certified SAM implements WebAuthn-derived SAD today could not be confirmed **[unverified]**; no certified deployment was found by name. This is a concrete, near-term PassSign opportunity that requires no new signature format at all.

---

## 9. Limitations Register

| # | Limitation | Severity | Layer |
|---|---|---|---|
| L1 | Authenticator signs only authData‖SHA-256(clientDataJSON); cannot sign DER(SignedAttributes)/JWS input | Blocking (until #2078) | WebAuthn |
| L2 | #2078 sign extension: draft PR, prototype-only, no WG implementation commitments, no ship date [10] | High | WebAuthn |
| L3 | No X.509 certificate for passkey keys; CSR PoP unachievable with plain credentials (RFC 2986) [17] | Blocking for AdES | PKI |
| L4 | Platform passkeys often return attestation `none`, weakening attestation-based issuance **[unverified currency]** | High | Platform |
| L5 | Synced passkeys replicate private keys across devices/cloud → "sole control" question for AdES/QES levels | High (legal, → Stream 4) | Platform/legal |
| L6 | No AlgorithmIdentifier OID / JOSE alg for assertion-format signatures; unknown algs → INDETERMINATE/FAILED in all validators | Blocking for interop | ETSI/IETF |
| L7 | No revocation model for passkey credentials → LTV semantics undefined without a certificate | Medium | AdES |
| L8 | Adobe/AATL & EUTL acceptance requires listed roots + recognized formats; multi-year onboarding | High | Ecosystem |
| L9 | Per-RP credential scoping conflicts with cross-context signing identity (privacy-by-design vs. legal identity) | Medium | Architecture |
| L10 | No ETSI/W3C joint work item on WebAuthn signature containers found **[unverified as exhaustive]** | Opportunity/risk | Standards |

---

## 10. Opportunities for PassSign

1. **Track A (ship now, fully standard output):** passkey-authorized remote signing — passkey as OAuth2/SAD factor over CSC API v2.2 toward an RSSP-held certified key. Output is byte-perfect PAdES/CAdES; validates in Adobe/DSS today. Differentiator: specify the WebAuthn-assertion-derived SAD (challenge = DTBS/R hash) as a cryptographic SCAL2 binding — apparently unspecified anywhere; a candidate contribution to ETSI TS 119 432 / CSC.
2. **Track B (standardization play):** author the missing profile — a JAdES-first "WebAuthn-ES256" algorithm (JOSE registration + ETSI TS 119 182 extension) carrying authenticatorData/clientDataJSON in `etsiU`/unprotected headers, plus a CMS twin (new OID + attributes) and a PAdES SubFilter. Section 4.2 shows the design is straightforward; the moat is process, not cryptography. Prior-art vacuum (Section 4.3) means first-mover framing is available.
3. **Track C (contingent on #2078):** if the sign extension ships, passkey authenticators emit raw signatures → only the certificate/profile/trust-list work remains. Position PassSign as the AdES profile + CA issuance policy (attestation-based, ACME device-attest-01-style) for sign-extension keys. Engaging in the PR #2078 / yubicolabs design loop now is cheap and buys influence.
4. **Certificate issuance for passkeys** is a separable, valuable work item regardless of track: define WebAuthn-assertion PoP and attestation-based PoP for ACME/EST — small specs, broad utility.
5. **RFC 3161 composes cleanly today** — any PassSign format, even proprietary-interim, should timestamp assertions immediately to accrue evidentiary value.

*This analysis addresses technical standards feasibility only; legal effect of signature formats (eIDAS qualification, sole-control interpretation, enforceability) is Stream 4's scope, and nothing here constitutes legal advice.*

---

## 11. Sources

1. RFC 5652, Cryptographic Message Syntax, §5.3–5.5, §11 — https://www.rfc-editor.org/rfc/rfc5652
2. Nutrient, "PDF digital signature standards: PAdES & CAdES" — https://www.nutrient.io/guides/web/signatures/digital-signatures/standards/
3. eID Easy, "PDF PAdES digital signatures using ETSI.CAdES.detached" — https://eideasy.com/pdf-pades-external-digital-signatures-using-etsi-cades-detached/
4. RFC 7515, JSON Web Signature, §5.1 — https://www.rfc-editor.org/rfc/rfc7515
5. ETSI EN 319 122-1 (CAdES) — https://www.etsi.org/deliver/etsi_en/319100_319199/31912201/
6. ETSI EN 319 142-1 (PAdES) — https://www.etsi.org/deliver/etsi_en/319100_319199/31914201/
7. ETSI TS 119 182-1 (JAdES) — https://www.etsi.org/deliver/etsi_ts/119100_119199/11918201/
8. ETSI EN 319 102-1 (signature creation & validation procedures) — https://www.etsi.org/deliver/etsi_en/319100_319199/31910201/
9. W3C Web Authentication Level 3, §6.3.3, §7.2 — https://www.w3.org/TR/webauthn-3/
10. w3c/webauthn PR #2078, "Add 'sign' extension" (emlun) — https://github.com/w3c/webauthn/pull/2078 (fetched; quotes verified)
11. Yubico, "Using WebAuthn for Signing" — https://developers.yubico.com/WebAuthn/Concepts/Using_WebAuthn_for_Signing.html (fetched)
12. Yubico/webauthn-sign-kem-extensions (draft) — https://github.com/Yubico/webauthn-sign-kem-extensions ; successor: https://github.com/yubicolabs/webauthn-sign-extension
13. FIDO Alliance, "Using FIDO with eIDAS Services" (2020) — https://fidoalliance.org/wp-content/uploads/2020/06/FIDO_Using-FIDO-with-eIDAS-Services-White-Paper.pdf
14. FIDO Alliance, "Deploying FIDO2 with eIDAS QTSPs / eID schemes" (2020) — https://fidoalliance.org/wp-content/uploads/2020/04/FIDO-deploying-FIDO2-eIDAS-QTSPs-eID-schemes-white-paper.pdf
15. RFC 3161, Time-Stamp Protocol — https://www.rfc-editor.org/rfc/rfc3161
16. Bicakci et al., "QES-Backed Virtual FIDO2 Authenticators," arXiv:2601.06554 (Jan 2026) — https://arxiv.org/abs/2601.06554 (fetched; abstract verified)
17. RFC 2986, PKCS #10 Certification Request Syntax — https://www.rfc-editor.org/rfc/rfc2986
18. J. Field, "WebAuthn for PKI professionals," goodsignin.com — https://www.goodsignin.com/blog/webauthn-for-pki-professionals (fetched)
19. awesome-webauthn (negative finding, prior run) — https://github.com/herrjemand/awesome-webauthn
20. EU DSS (Digital Signature Services) library — https://ec.europa.eu/digital-building-blocks/sites/display/DIGITAL/eSignature ; source: https://github.com/esig/dss
21. Adobe Approved Trust List — https://helpx.adobe.com/acrobat/kb/approved-trust-list1.html
22. Cloud Signature Consortium, CSC API V2.2 page — https://cloudsignatureconsortium.org/resources/csc-api-v2-2/
23. CSC API v2.2 specification PDF (Nov 2025) — https://cloudsignatureconsortium.org/wp-content/uploads/2025/11/csc-api.pdf
24. ETSI TS 119 432 V1.3.1 (2026-03) — https://www.etsi.org/deliver/etsi_ts/119400_119499/119432/01.03.01_60/ts_119432v010301p.pdf
25. EUDI standards tracker, issue #68 (TS 119 432 status, EN planned 2027) — https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/issues/68
26. Entrust, "Signature Activation Module" service description (EN 419 241-2/SCAL2 context) — https://www.entrust.com/sites/default/files/documentation/service-descriptions/signature-activation-module-sd.pdf
