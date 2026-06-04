---
id: PP-20260604-9da905
title: "ARC42STORIES.MD requires all source material before writing — not just CLAUDE.md"
type: rule
scope: repo
applies_to: "Any session writing or updating ARC42STORIES.MD for a casehub module"
severity: important
refs:
  - docs/DESIGN.md
  - docs/DESIGN-capabilities.md
  - docs/CAPABILITIES.md
  - docs/adr/
created: 2026-06-04
violation_hint: "ARC42STORIES.MD contains wrong class names, stale terminology, missing capabilities, or incorrect field names — sourced only from CLAUDE.md"
---

Before writing or substantially updating `ARC42STORIES.MD` for any casehub module, read all available source material: every blog entry in the workspace `blog/` directory, `docs/DESIGN.md`, `docs/DESIGN-capabilities.md`, all ADRs in `docs/adr/`, any design specs in the workspace `specs/` directory, and the integration guide. Writing from CLAUDE.md alone — even a thorough CLAUDE.md — produces factual errors and material omissions: in the initial write of `casehub-ledger`'s ARC42STORIES.MD this produced 11 factual errors (wrong class names, wrong field names, wrong pass counts, wrong minimum version numbers) and 15 significant omissions (missing capabilities, missing gotchas, missing ADR decisions). The blogs are the primary narrative source for `§9 Journeys and Chapters`; the design docs and ADRs are the authority for `§9.4 Architectural decisions` and the `§13 Glossary`.
