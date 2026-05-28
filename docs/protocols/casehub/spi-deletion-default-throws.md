---
id: PP-20260528-557f5c
title: "SPI defaults for data-deletion operations must throw, not silently no-op"
type: rule
scope: platform
applies_to: "All casehub-platform-api SPI interfaces with erasure or deletion methods"
severity: important
refs:
  - docs/protocols/universal/module-tier-structure.md
violation_hint: "A default eraseById() or delete() that returns void silently — gives caller a success signal with no deletion performed"
created: 2026-05-28
---

When a SPI interface provides a `default` implementation for a method that deletes, erases, or removes data, the default MUST throw `UnsupportedOperationException` — not return silently. A silent default on a deletion method gives the caller a false success signal: the operation appears to succeed but no data is removed. The real no-op override belongs only on the concrete `@DefaultBean` implementation (e.g. `NoOpCaseMemoryStore`) where the semantics are correct: nothing was stored, so nothing can be erased. The interface default's job is to signal "this adapter does not support this operation" loudly. This is especially important for GDPR-adjacent erasure paths where a silent failure is legally significant. Example: `CaseMemoryStore.eraseById()` defaults to `throw new UnsupportedOperationException("eraseById not supported by this adapter")`, while `NoOpCaseMemoryStore.eraseById()` overrides with `{}`.
