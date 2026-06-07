---
id: PP-20260607-916fc4
title: "@VocabularyMetadata annotation required on all VocabularyTerm enum classes"
type: rule
scope: repo
applies_to: "casehub-eidos: any enum implementing VocabularyTerm; any module registering vocabularies via VocabularyRegistrar"
severity: important
refs:
  - casehubio/eidos#40
violation_hint: "VocabularyRegistrar bean returning a class without @VocabularyMetadata; runtime IllegalArgumentException: missing @VocabularyMetadata at startup"
created: 2026-06-07
---

Any Java enum implementing `VocabularyTerm` must carry `@VocabularyMetadata(uri = "urn:...")`. `CdiVocabularyRegistry.register(Class<T>)` reads the annotation via reflection at startup and throws `IllegalArgumentException` if absent. A missing annotation produces a runtime failure rather than a compile error — formalising the contract allows vocabulary authors to catch it in code review rather than at startup.
