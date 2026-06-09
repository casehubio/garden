---
id: PP-20260609-9b403d
title: "ERASE_BY_ID capability requires complete erasure — source record plus all derived data"
type: rule
scope: platform
applies_to: "All CaseMemoryStore adapter implementations declaring MemoryCapability.ERASE_BY_ID"
severity: critical
refs:
  - platform-api/src/main/java/io/casehub/platform/api/memory/CaseMemoryStore.java
  - platform-api/src/main/java/io/casehub/platform/api/memory/MemoryCapability.java
violation_hint: "Declaring ERASE_BY_ID when eraseById() only deletes the source episode/record but leaves LLM-extracted entity nodes, relationship edges, or other derived facts intact — as in Graphiti's DELETE /episode/{uuid} which removes the EpisodicNode but not derived EntityNode/EntityEdge records"
created: 2026-06-09
---

An adapter may only include `MemoryCapability.ERASE_BY_ID` in its `capabilities()` return set
if the `eraseById()` implementation guarantees complete deletion of all data derived from
or associated with that memory ID — including any facts, entities, relationships, or embeddings
extracted from the original stored text. Partial deletion (source record gone, derived knowledge
graph entries persist) does not satisfy GDPR Art.17 right-to-erasure semantics and must not be
presented as erasure to callers. Adapters where the underlying backend does not support cascade
deletion must remove `ERASE_BY_ID` from `capabilities()` and throw `MemoryCapabilityException`
from `eraseById()` until the backend adds the necessary support. Track the upstream gap as an
issue linked to the capability declaration. Use `eraseEntity()` (which typically issues
group/entity-scoped bulk deletion) as the alternative for GDPR-safe erasure when per-memory
deletion is insufficient.
