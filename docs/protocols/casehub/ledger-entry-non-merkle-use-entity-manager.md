---
id: PP-20260531-5b4fee
title: "LedgerEntry subclasses outside the Merkle audit chain must persist via EntityManager, not LedgerEntryRepository.save()"
type: rule
scope: application
applies_to: "casehub harnesses writing LedgerEntry subclasses that should not appear in the Merkle-chained audit trail (e.g. internal attestation records, routing metadata)"
severity: important
refs:
  - casehub-ledger/GE-20260531-d2ed26 (garden: LedgerEntryRepository.save() triggers Merkle chain update)
violation_hint: "UQ_MERKLE_FRONTIER_SUBJECT_LEVEL constraint violation when two or more LedgerEntry subclasses are saved concurrently for the same subjectId"
created: 2026-05-31
---

`LedgerEntryRepository.save()` triggers the `LedgerTraceListener.prePersist()` → `LedgerEnricherPipeline` → Merkle frontier update for every entity persisted. When parallel engine workers write `LedgerEntry` subclasses for the same `subjectId` simultaneously, concurrent Merkle writes violate `UQ_MERKLE_FRONTIER_SUBJECT_LEVEL`. For `LedgerEntry` subclasses that are internal records (routing attestations, diagnostic metadata) and should NOT participate in the externally-verifiable Merkle chain, persist directly via `@PersistenceContext EntityManager em; em.persist(entry)` — this bypasses the enricher pipeline entirely. Only use `LedgerEntryRepository.save()` for entities where Merkle inclusion and the full enrichment lifecycle (actor identity tokenisation, trace ID propagation) are required.
