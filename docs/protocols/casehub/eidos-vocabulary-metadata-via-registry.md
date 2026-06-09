---
id: PP-20260609-908111
title: "Renderer enrichment must read vocabulary metadata through VocabularyRegistry, not directly from annotation"
type: rule
scope: repo
applies_to: "casehub-eidos runtime renderer: any code in EidosRenderPipeline or its helpers that needs vocabulary-level metadata (name, description, uri)"
severity: important
refs:
  - docs/superpowers/specs/2026-06-08-framework-grounding-design.md
violation_hint: "SomeVocabEnum.class.getAnnotation(VocabularyMetadata.class); enum.getDeclaringClass().getAnnotation(...); reading @VocabularyMetadata outside of CdiVocabularyRegistry"
created: 2026-06-09
---

Pipeline enrichment code that needs vocabulary-level metadata (name, description, URI) must call `VocabularyRegistry.vocabularyMetadata(uri): Optional<VocabularyMetadata>`, never read the `@VocabularyMetadata` annotation directly from an enum class instance. The registry is the authorised abstraction layer: it holds the byUri map, enforces the registration invariant, and returns `Optional.empty()` for unknown URIs without NPE risk. Direct annotation reads bypass registration state, couple the renderer to the enum class hierarchy, and silently succeed for unregistered vocabularies. The only code that reads `@VocabularyMetadata` annotations directly is `CdiVocabularyRegistry.register()` and `CdiVocabularyRegistry.vocabularyMetadata()` — all other call sites are below the abstraction boundary.
