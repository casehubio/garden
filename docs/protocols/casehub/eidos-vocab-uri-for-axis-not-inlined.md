---
id: PP-20260607-af47ac
title: "Use vocabUriForAxis() — do not inline the disposition axis vocabulary resolution chain"
type: rule
scope: repo
applies_to: "casehub-eidos consumers resolving vocabulary URIs for AgentDescriptor disposition fields; any code calling VocabularyRegistry with an AgentDescriptor"
severity: guidance
refs:
  - ../../repos/casehub-eidos.md
violation_hint: "Call site contains axisVocabularies.get(axis) or conditional checks against dispositionVocabulary/domainVocabulary rather than calling descriptor.vocabUriForAxis(axis)"
created: 2026-06-07
---

Callers resolving the vocabulary URI for a disposition axis must call `AgentDescriptor.vocabUriForAxis(DispositionAxis)`. The method encapsulates the three-step resolution precedence: `axisVocabularies.get(axis)` → `dispositionVocabulary` → `domainVocabulary` → `Optional.empty()`. Inlining the chain at call sites means each site independently implements a precedence rule that may drift from the authoritative version as the descriptor evolves. The canonical integration pattern is: `descriptor.vocabUriForAxis(axis).flatMap(uri -> registry.resolve(uri, value))` for axis-aware resolution, and `registry.equivalentValues(from, targetVocab, axis)` for typed cross-vocab lookups.
