---
id: PP-20260605-232544
title: "DISC types are disposition vocabulary — never slot vocabulary"
type: rule
scope: platform
applies_to: "casehub-eidos-vocab vocabulary producers; any code setting slotVocabulary or dispositionVocabulary on AgentDescriptor"
severity: important
refs:
  - casehub-eidos/docs/personality-frameworks.md
violation_hint: "DiscVocabularyProducer or AgentDescriptor construction sets slotVocabulary to the DISC vocabulary URI, or uses a DISC type name (dominance, influence, steadiness, conscientiousness-disc) as a slot value"
created: 2026-06-05
---

DISC types (Dominance, Influence, Steadiness, Conscientiousness) describe behavioral patterns an agent brings to every context — they are not roles assigned by a team. Register DISC as `dispositionVocabulary`, not `slotVocabulary`.

An agent can hold both simultaneously: `slotVocabulary=belbin` (the team role assignment) and `dispositionVocabulary=disc` (the behavioral style). The inverse — `slot="dominance"` — treats a personality pattern as a role assignment, which is a category error. It also prevents the agent from holding both a Belbin role and a DISC style simultaneously.

See `docs/personality-frameworks.md §Architecture: DISC as Disposition Vocabulary` in casehub-eidos.

Established by eidos#29. Prerequisite for the Belbin/DISC vocabulary module (eidos#26).
