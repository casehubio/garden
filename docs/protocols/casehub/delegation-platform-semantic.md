---
id: PP-20260605-1000ad
title: "delegation is platform-semantic — personality framework types make no delegation claim"
type: rule
scope: platform
applies_to: "AgentDescriptor construction; disposition annotation in personality vocabulary producers (Belbin, DISC, MBTI, etc.)"
severity: important
refs:
  - casehub-eidos/docs/personality-frameworks.md
violation_hint: "VocabularyTerm annotation or AgentDescriptor construction sets delegation=true based on a personality framework characteristic — e.g. DISC Dominance delegates=true because D-types assign tasks, or MBTI ENTJ implies delegation"
created: 2026-06-05
---

The boolean `delegation` field means 'can spawn sub-agents' — a platform capability, not a personality trait. Personality frameworks (DISC, MBTI, Big Five, Belbin) describe behavioral style, not platform affordances.

DISC D-types assign tasks but may maintain tight oversight; this does not predict sub-agent spawning. Only Belbin roles with explicit team-empowerment semantics (Co-ordinator) warrant `delegation=true`, and this is derived from the role definition, not from a personality axis.

Personality vocabulary producers (`DiscVocabularyProducer`, `BelbinVocabularyProducer`) must not annotate `VocabularyTerm` entries with delegation values. `delegation` is set directly on `AgentDescriptor` by the agent's owner based on platform capability, not vocabulary resolution.

See `docs/personality-frameworks.md §Anti-Patterns` in casehub-eidos.

Established by eidos#29.
