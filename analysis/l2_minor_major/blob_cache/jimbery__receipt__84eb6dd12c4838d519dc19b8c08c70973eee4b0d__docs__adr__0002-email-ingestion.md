# ADR-002: Email Receipt Ingestion Architecture

**Status:** Accepted  
**Date:** 2026-06-10  
**Phase:** 1 — Email receipt ingestion  
**Deciders:** Jay  
**Depends on:** [ADR-001](0001-record-architecture-decisions.md) (canonical `Receipt`, matching contract)

---

## Context

Phase 1 turns volunteer mailboxes into canonical `Receipt` records with completeness grading, deduplication, and SA categorisation — without per-receipt user action. The moat metric is **cohort-weighted purchase coverage**: fraction of confirmed business purchases captured at `itemised` grade, weighted by Track A merchant mix.

Constraints:

- **C1** — no merchant-cooperation dependency; extract from whatever the mailbox contains.
- **C2** — itemisation rate is the moat; precision over coverage (garbage receipts poison matching).
- **ADR-001 downstream contract** — duplicate receipts must not be silently merged; ambiguous families defer to matcher `conflict`.
- **Pilot only** — Gmail/Microsoft Graph behind interfaces; production OAuth verification deferred until gate passes.
- **No LLM extraction** in Phase 1 (revisit trigger: priority merchants still below field-accuracy gate after one tuning cycle).

## Decision

### D1 — Layered pipeline: source → classify → extract → grade → canonicalise → deduplicate

1. **Mail source** — `MailSource` fetches `RawMessage` records (Maildir for tests; Gmail/Graph for pilot).
2. **Classifier** — assigns `DocumentKind` before extraction cost is spent.
3. **Extractor registry** — merchant-specific extractors + generic fallbacks; pluggable without changing pipeline wiring (C2).
4. **Grading** — shared `CompletenessGrade` function on `ExtractedReceipt`.
5. **Canonicaliser** — maps to ADR-001 `Receipt` with arithmetic validation.
6. **Deduplication** — purchase-family resolution; low-confidence ambiguity emits multiple records.

All stages are deterministic for identical inputs + config (same discipline as ADR-001 D4).

### D2 — Classifier (D2.1 rule-based)

Rule-based classifier over sender domain, subject patterns, and MIME structure. Outputs `DocumentKind`:

| Kind | Treatment |
|---|---|
| `purchase_receipt` | extract |
| `credit_note` | extract, retain for Phase 2 refund matching |
| `dispatch_notice` | extract envelope-grade only if no better sibling |
| `marketing` | skip |
| `statement` | skip |
| `unknown` | attempt generic extractor; grade conservatively |

Credit notes are **never dropped**. Hard negatives ("your receipt awaits", dispatch-without-total) have committed fixture patterns.

### D3 — Completeness grading

Shared grade function on extracted fields:

| Grade | Criteria (summary) |
|---|---|
| `itemised` | supplier + date + total reconcile; ≥1 line item with VAT or explicit VAT-exempt; arithmetic within tolerance |
| `partial` | supplier + date + total present but lines/VAT incomplete or arithmetic mismatch |
| `envelope` | purchase identity (supplier, date, total) without line detail |
| `requires_ocr` | image-only PDF or no machine-readable body |

Arithmetic mismatch **demotes** `itemised` → `partial` (never fabricate fields).

### D4 — Purchase families and Amazon shipment rule

**Family key** — prefer merchant order reference; fallback fingerprint (supplier + date + total minor + currency).

**Precedence** — invoice > shipment confirmation > marketing envelope for same family.

**Amazon** — one order may produce **N shipment-level receipts** (one charge per shipment). Order confirmation alone does not collapse shipments.

**Low-confidence family** — when refs absent and fingerprint ambiguous, emit **both** records (matcher `conflict` is the safe contract).

### D5 — Volunteer data, consent, and fixture hygiene

- Consent template + data-handling doc committed **before** any OAuth token.
- Volunteer ground truth: human-labelled purchases per mailbox (target ≥40 per volunteer; flag if <25).
- **PII scrubbing tool** for fixtures: pattern-based redaction (UK postcodes, greetings, addresses, order refs) plus per-mailbox dictionary; property-tested via `DetectPIIViolations` and signed `manifest.json` for `.eml` fixtures.
- Real mailbox content never committed.

### D6 — Ingestion harness

**Fixture-lab gate** (`ingest-fixture-smoke-v2`): classifier precision/recall on frozen corpus, `requires_ocr` count, extractor/classifier hashes. Enforced in CI. **Phase 1 exit criterion** as ratified 2026-06-11.

**Moat gate (transferred to Phase 2):** cohort-weighted purchase coverage, paper-gap quantification, kill/pivot decision tree — originally M1.6 exit; relocated per [2026-06-11 ratification](../validation/phase-01/2026-06-11-ratification-gate-deferral.md).

Per-merchant field accuracy and dedup correctness are enforced by table tests (M1.4–M1.5).

### D7 — Moat metric (Phase 2 target; transferred from M1.6)

**Cohort-weighted purchase coverage** = Σ(weight_m × itemised_purchases_m) / Σ(weight_m × confirmed_purchases_m).

Originally the M1.6 exit gate; **transferred to Phase 2** per [2026-06-11 ratification](../validation/phase-01/2026-06-11-ratification-gate-deferral.md). Thresholds (≥70% combined, ≥80% email-channel) remain committed; measurement deferred until volunteer mailboxes and Open Banking ingestion exist.

Decision tree (Phase 2 moat gate):

1. Email-channel coverage OK **and** combined projection OK → proceed Phase 3 normalisation.
2. Email OK, combined low, paper-gap ≥40% of misses → accelerate Phase 4 OCR.
3. Email-channel low → extraction diagnosis; one tuning cycle; else pivot.

## Consequences

**Positive**

- Maildir + scrubbed fixtures enable full CI without OAuth.
- Extractor registry satisfies C2 pluggability.
- Dedup integration test discharges ADR-001 duplicate-receipt contract at the seam.

**Negative / trade-offs**

- Rule classifier will miss long-tail merchants until corpus grows.
- Pilot OAuth stubs require manual token setup outside CI.
- LLM deferred — some PDF layouts will route to `requires_ocr` until Phase 4.

## Test implications

- Maildir round-trip; scrubber property tests; classifier corpus table + fuzz MIME parser.
- Grade function edge cases; canonicaliser arithmetic property test.
- Per-merchant extractor fixtures; dedup outcome table (M1.5); matcher integration on duplicate low-confidence.
- Fixture-lab gate on frozen corpus (`make evaluate-ingest-smoke`).

## Amendment 2026-06-10 — Gate thresholds and cohort weights (ratified)

Jay Imbery ratified Phase 1 fixture-lab thresholds and the provisional MTD cohort weight vector.

| Item | Committed value | Anchor |
|---|---|---|
| Classifier precision / recall (receipt-bearing) | ≥ 97% / ≥ 92% | `DefaultSmokeConfig()` |
| Field accuracy (total/date/supplier) | ≥ 98% | `merchant_field_accuracy_test` |
| Dedup false merges | 0 | M1.5 fixture table |
| Cohort weights (corpus prioritisation) | Screwfix 0.18, Toolstation 0.12, B&Q 0.10, Amazon 0.15, Shell 0.08, BP 0.05 | `test/testdata/cohort_weights.json` |

Moat thresholds (≥70% combined, ≥80% email-channel) remain committed for **Phase 2 moat gate** — not measured in Phase 1.

Gate deferral ratified 2026-06-11: [2026-06-11-ratification-gate-deferral.md](../validation/phase-01/2026-06-11-ratification-gate-deferral.md).

Full ratification: [docs/validation/phase-01/2026-06-10-ratification-thresholds.md](../validation/phase-01/2026-06-10-ratification-thresholds.md).

## Amendment 2026-06-11 — Moat gate transferred to Phase 2 (ratified)

Jay Imbery ratified formal deferral of the volunteer moat gate from Phase 1 M1.6 to Phase 2. Phase 1 closes on fixture-lab CI evidence only. Accepted cost: kill/pivot decision tree and paper-gap measurement fire later, after Open Banking ingestion begins.

See [2026-06-11-ratification-gate-deferral.md](../validation/phase-01/2026-06-11-ratification-gate-deferral.md).

## Alternatives considered

| Option | Why not |
|---|---|
| LLM-first extraction | No training/eval baseline; precision risk; deferred with revisit trigger |
| Ingest inside matcher package | Violates ADR-001 source-agnostic boundary |
| Merge ambiguous families silently | Violates ADR-001; downstream tax-record risk |
