---
id: PP-20260607-318a05
title: "CdiVocabularyRegistry.register() is @PostConstruct-only — not thread-safe after startup"
type: rule
scope: repo
applies_to: "casehub-eidos: any caller of VocabularyRegistry.register(Class<T>)"
severity: important
refs:
  - casehubio/eidos#40
violation_hint: "Calling registry.register() from @Observes, @Scheduled, or request-scoped beans after application startup"
created: 2026-06-07
---

`CdiVocabularyRegistry.register()` modifies three internal `ConcurrentHashMap` instances in sequence without an atomic lock. Concurrent calls can interleave: two simultaneous calls could both pass the duplicate-URI check before either writes the URI key. The method is designed for single-threaded initialization via `@PostConstruct`. After `@PostConstruct` completes, concurrent reads are safe (ConcurrentHashMap visibility guarantees apply). Programmatic `register()` calls from application code after startup must be externally synchronized or prohibited.
