---
id: PP-20260619-409a36
title: "Inject Instance<T> with isUnsatisfied() for optional platform SPIs — never add a competing @DefaultBean"
type: rule
scope: platform
applies_to: "Any casehub extension or library module that optionally depends on a platform-owned SPI (e.g. PreferenceProvider, AgentProvider)"
severity: important
refs:
  - ../../../docs/PLATFORM.md
violation_hint: "Extension provides a @DefaultBean implementation of a platform-owned type; AmbiguousResolutionException fires at augmentation when casehub-platform is also on the classpath"
garden_ref: "GE-20260601-fcf0d9"
created: 2026-06-19
---

When a casehub extension module wants to make a platform-owned SPI optional (i.e. the extension works with or without it), inject `Instance<T>` and guard with `isUnsatisfied()` at call time — do not declare a `@DefaultBean` implementation of the platform type in the extension. Two `@DefaultBean` beans of the same type — one from the extension, one from `casehub-platform` (e.g. `MockPreferenceProvider`) — cause `AmbiguousResolutionException` at Quarkus augmentation time because neither can displace the other; the pattern to use is: `@Inject Instance<PreferenceProvider> pp; if (pp.isUnsatisfied()) { use default; } else { pp.get().resolve(...); }`.
