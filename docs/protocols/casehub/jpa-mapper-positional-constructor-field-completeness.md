---
id: PP-20260608-e694ab
title: "Full-fidelity JPA-to-record mappers must use the positional constructor, not Builder"
type: rule
scope: repo
applies_to: "casehub-eidos AgentDescriptorMapper.toRecord() and any future full-fidelity JPA-to-record mapper"
severity: important
refs:
  - docs/superpowers/specs/2026-06-07-vocab-registry-polish-design.md
violation_hint: "AgentDescriptorMapper.toRecord() replaced with AgentDescriptor.builder()...build() — a new AgentDescriptor field added later will silently receive null instead of failing to compile"
created: 2026-06-08
---

`AgentDescriptorMapper.toRecord()` must use `new AgentDescriptor(...)` (positional constructor), not `AgentDescriptor.builder()...build()`. The positional constructor provides compile-time field-completeness enforcement: when a new field is added to `AgentDescriptor`, the mapper will not compile until the new field is explicitly sourced from the entity. A Builder call silently defaults any unset field to null — a full-fidelity JPA→record mapping where a new field is silently null is a correctness bug. The positional constructor must remain non-private for this reason. This is the only production call site with this property; test code uses the Builder throughout.
