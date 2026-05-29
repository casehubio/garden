---
id: PP-20260529-4783b2
title: "Ledger sequence numbers must be computed across all LedgerEntry subtypes via findLatestBySubjectId()"
type: rule
scope: platform
applies_to: "casehub-engine-ledger and any module adding a new LedgerEntry JOINED subclass"
severity: critical
refs:
  - docs/protocols/casehub/harness-ledger-writer.md
violation_hint: "Observer uses findLatestByCaseId() (or any subtype-scoped query) to compute the next sequence number — misses sibling LedgerEntry subtypes for the same subjectId, silently producing duplicate sequence numbers and a corrupt Merkle chain."
created: 2026-05-29
---

When computing the next `sequenceNumber` for a `LedgerEntry` write, always use
`findLatestBySubjectId(subjectId)` — which queries the `LedgerEntry` base class and
spans all JOINED subtypes — not a subtype-scoped query such as `findLatestByCaseId()`.
The sequence and Merkle chain are scoped to `subjectId` across all entry types; if any
sibling subtype holds a higher sequence for the same `subjectId`, a subtype-scoped query
will return a stale max and the new entry will collide. Discovered in engine#390: adding
`WorkerDecisionEntry` alongside `CaseLedgerEntry` for the same caseId required fixing
`CaseLedgerEventCapture` from `findLatestByCaseId()` to `findLatestBySubjectId()`.
