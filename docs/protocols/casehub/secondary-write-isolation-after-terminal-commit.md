---
id: PP-20260607-697a78
title: "Secondary audit writes after a terminal-status REQUIRES_NEW must run outside the primary try-catch"
type: rule
scope: application
applies_to: "Any harness service that commits entity state with REQUIRES_NEW and then writes secondary audit/ledger entries"
severity: critical
refs:
  - ../../repos/casehub-clinical.md
violation_hint: "A secondary ledger write (e.g. deviationLedgerWriter.writeSponsorNotifiedEntry) sits inside the same try-catch as the connector call and store.markDelivered(). If the secondary write throws, the catch block calls store.markFailed() — downgrading a committed DELIVERED entity back to FAILED. The scheduler retries and the sponsor receives a duplicate notification."
garden_ref: "GE-20260607-0bfc83"
created: 2026-06-07
---

When a `@Transactional(REQUIRES_NEW)` call commits an entity to a terminal state (DELIVERED, EXHAUSTED, COMPLETED), any subsequent secondary writes — deviation ledger entries, audit chain entries, or cross-datasource writes — must execute **outside** the try-catch that guards the primary action. Each secondary write gets its own isolated `try { ... } catch (Exception e) { LOG.errorf(...); }` that logs on failure without modifying the already-committed entity state. Add a terminal-status guard at the top of every REQUIRES_NEW entity update method (`if (n.status == TERMINAL_A || n.status == TERMINAL_B) return;`) to make this invariant enforceable from any call site.
