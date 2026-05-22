---
id: PP-20260522-359dfc
title: "CurrentPrincipal boolean classifier methods must delegate to actorType()"
type: rule
scope: platform
applies_to: "casehub-platform-api — CurrentPrincipal interface and all implementors"
severity: important
refs:
  - docs/protocols/casehub/platform-spi-contract.md
violation_hint: "isSystem() or a similar boolean returns true/false via actorId string matching instead of comparing actorType() — diverges silently when new actorId formats are added"
created: 2026-05-22
---

`CurrentPrincipal` boolean methods that classify the actor type (`isSystem()`, and any future equivalents) must delegate to `actorType()` rather than implementing independent actorId pattern matching. An exact-match or prefix check on the actorId string diverges from `ActorTypeResolver`'s resolution rules as soon as new actorId formats are added — for example, `"system".equals(actorId())` returns `false` for `system:scheduler` while `actorType()` correctly returns `SYSTEM`. `actorType()` is the single source of truth for actor classification; all boolean derivatives must agree with it. Default: `default boolean isSystem() { return actorType() == ActorType.SYSTEM; }`.
