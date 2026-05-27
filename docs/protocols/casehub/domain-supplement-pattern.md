---
id: PP-20260527-da1f66
title: "Attach domain context to foundation primitives via a supplement table — not a wrapper entity"
type: principle
scope: application
applies_to: "Application-tier repos (life, aml, clinical, devtown) that need to attach domain-specific metadata to WorkItem or CaseInstance"
severity: important
refs:
  - docs/protocols/casehub/ledger-subclass-extension.md
  - docs/AGENTIC-HARNESS-GUIDE.md
violation_hint: "App creates a full domain entity (HouseholdTask) that duplicates the foundation primitive and adds a workItemId FK — the entity becomes redundant at Layer 5 when the foundation primitive IS the coordination record"
created: 2026-05-27
---

When a domain application needs domain-specific context alongside a foundation primitive (WorkItem, CaseInstance), use an optional supplement table — not a wrapper entity. The supplement table is owned and migrated by the consuming application; the foundation primitive knows nothing about it. A supplement is justified when the domain context is too thin for a full domain entity (a single FK to ExternalActor, a legal case reference, a trial ID) but too domain-specific to belong in the foundation primitive's generic payload field. This pattern is distinct from ledger subclass extension (JOINED inheritance for extending the tamper-evident audit chain) — supplements are for lightweight domain context, not compliance chain extension. A wrapper entity that re-exposes foundation primitive fields plus domain additions is never correct: at higher layers the foundation primitive becomes the coordination record and the wrapper becomes redundant.
