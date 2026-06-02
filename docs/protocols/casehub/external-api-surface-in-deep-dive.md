---
id: PP-20260602-e36824
title: "Document new API surface in the repo deep-dive, not DESIGN.md"
type: rule
scope: platform
applies_to: "All casehub peer repos when adding new public types, SPIs, or services to an api/ or runtime/ module"
severity: important
refs:
  - casehub/FOUNDATION-INDEX.md
violation_hint: "New SPI, DTO, record, or @ApplicationScoped service ships in a repo but appears only in DESIGN.md or not at all — not in docs/repos/casehub-{repo}.md Key Abstractions or Module Structure"
created: 2026-06-02
---

When a casehub peer repo adds new API surface — a new SPI interface, DTO, record, or service class — it must be documented in the repo's deep-dive (`docs/repos/casehub-{repo}.md` in the parent repo) before or alongside the PR that ships it. DESIGN.md in the repo is ephemeral session context; the deep-dive in parent is the durable, cross-session reference. Consumers discovering a new type should find it by reading the deep-dive, not by searching the codebase. The Module Structure table documents which module the type lives in; Key Abstractions documents its contract and semantics.
