# Hash-Chain Document Signing

A document-signing workflow with signed-PDF generation, completion certificates, per-page signature metadata, and an append-only event hash chain for tamper detection.

**Status:** Built as an internal signing system for a European construction SME. It is not presented as a qualified electronic signature system. The hash chain is a defense-in-depth integrity layer, not a substitute for a qualified trust service provider.

---

## Problem

The first version of the signing flow was functionally useful but not trustworthy enough. A customer could sign, the UI could show success, and yet a downstream PDF-generation failure could still leave the stored "signed" PDF missing, stale, or unsigned.

That is the dangerous class of bug: the system says the legal/commercial step succeeded, while the artifact does not prove it.

The goal became bigger than "capture a signature." The signing system needed:

- deterministic signed-PDF generation
- visible audit trail
- completion certificate
- per-page signature metadata
- explicit failure instead of silent success
- tamper detection for signing events
- bilingual customer-facing output

---

## Constraints that shaped the design

1. **Simple electronic signature scope.** Qualified e-signature via national identity providers was parked until credentials, contracts, and cost model were available.
2. **PDFs are the customer artifact.** The signed document has to be inspectable as a PDF, not only as database state.
3. **No silent fallback.** Serving an unsigned PDF after signing is worse than showing an error.
4. **Cloudflare runtime.** PDF generation, R2 storage, D1 events, and Web Crypto must work inside the deployed stack.
5. **Bilingual certificate text.** Dutch and French output are required.
6. **Tamper-evidence, not legal overclaiming.** The public and customer-facing language must avoid pretending a hash chain equals QES/PAdES.

---

## Architecture

```text
        Signing token created
                 |
                 v
        signing_events seq 1
        hash = SHA-256(genesis)
                 |
      Customer views / signs / declines
                 |
                 v
        Event producer wrapper
        append row with prev_hash
                 |
                 v
      Signed PDF generation pipeline
 original/wrapped PDF + signature image
 + master footer + completion certificate
                 |
                 v
          R2 signed artifact
          D1 signed_pdf_key
                 |
                 v
       Certificate renders chain status
```

The design separates the signed artifact from the event log. The PDF is the human-facing proof. The hash chain is the integrity check over the signing timeline.

---

## Key technical decisions

### 1. Throw on signing-generation failure

The original failure pattern was "return false and keep going." That is unacceptable for signing. The corrected rule is simple: if the signed artifact cannot be generated and stored, the signing request fails.

That creates a worse immediate user experience in rare failure cases, but a much safer system.

### 2. Prefer signed artifact everywhere

Once a signed PDF exists, customer and internal download endpoints should prefer it over wrapped or original PDFs. Fallbacks are useful before signing; after signing, fallback can become misinformation.

### 3. Completion certificate as a frozen snapshot

The certificate records envelope ID, signer, method, timestamps, device/IP metadata where available, event timeline, and disclosure text. It is generated from a snapshot so later database changes do not silently rewrite the historical certificate.

### 4. Linear hash chain over event rows

Each signing event stores:

- sequence number
- event type
- payload JSON
- previous hash
- current hash
- timestamp

The v1 hash input binds the previous hash, event type, payload, timestamp, token ID, and sequence number. Any mutation in the chain breaks verification from that point onward.

### 5. Optimistic retry for event concurrency

Two events can race. The chain uses a unique `(token_id, sequence_nr)` constraint. If two writers read the same tail, one insert wins, the other retries after reading the new tail.

This is simpler than introducing a queue and sufficient for human-paced signing events.

### 6. Honest legal framing

The hash chain can detect mutation. It does not make the signature qualified. That distinction is central. The certificate can say events are cryptographically linked; it should not say the result is legally unchallengeable.

---

## Outcomes

| Metric | Value |
|---|---:|
| Signing layers scoped | master footer, completion certificate, hash chain |
| Hash-chain scope | event integrity only |
| External QES/PAdES dependency | explicitly parked |
| Core storage | D1 for tokens/events, R2 for PDFs/signatures |
| Crypto primitive | SHA-256 via Web Crypto |
| Failure policy | throw and surface error, no silent success |

**Real win:** the system moved from "a signature was captured" to "the signed artifact, certificate, and event trail can be inspected together."

---

## Lessons

### Signing systems cannot tolerate silent success

Most app bugs are annoying. Signing bugs can create false legal confidence. If the final artifact is missing or wrong, the UI must not say success.

The product decision is brutal but correct: fail loudly.

### Tamper-evidence is useful even when it is not QES

A hash chain does not replace a qualified trust service. It still catches accidental mutation, partial tampering, and unsophisticated database edits. That is meaningful defense in depth as long as the language stays honest.

### The PDF is part of the state machine

It is not enough for database status to say `signed`. The R2 object, certificate snapshot, download endpoint, and visible footer all have to agree.

### Scope parking is architecture work

The qualified-signature path was deliberately parked because credentials, provider contracts, and per-signature economics were not ready. Building dead code around an unavailable provider would have made the system more complex without increasing trust.

---

## What's NOT in this case study

- Legal advice or claims of qualified electronic signature status
- Customer documents or certificate screenshots
- Internal token schemas beyond the architectural fields described above
- Provider credentials, vendor names, or private endpoint details
- Source code from the signing service

The reusable pattern is the integrity model: explicit failure, signed-artifact preference, frozen certificate snapshots, append-only event chains, and honest legal language.

---

**Built:** April 2026, after a signing-pipeline audit found silent-failure paths that made "success" untrustworthy.
