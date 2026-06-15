---
id: PP-20260615-9adaee
title: "No-op @DefaultBean implementations must not fire CDI events"
type: rule
scope: platform
applies_to: "Any @DefaultBean no-op implementation of EndpointRegistry, CaseMemoryStore, ActualStateAdapter, or any registry/store SPI in casehub-platform — whenever a @DefaultBean serves as the silent fallback"
severity: important
refs:
  - docs/PLATFORM.md
violation_hint: "A no-op registry fires Event<EndpointRegistered> in register() — Camel routes, Kafka subscriptions, or adapter configuration triggers for phantom endpoints that were never actually stored"
garden_ref: "GE-20260615-00ff7a"
created: 2026-06-15
---

No-op `@DefaultBean` implementations of registry and store SPIs — such as `NoOpEndpointRegistry`, `NoOpCaseMemoryStore`, or any future no-op `ActualStateAdapter` — must remain completely silent. They must not fire CDI events that observers use to trigger downstream self-configuration. A no-op signals "nothing exists here": firing events would cause platform stream modules to start Camel routes, Kafka subscriptions, or other adapters for phantom endpoints or entries that were never actually stored. Only implementations that genuinely store the value — `InMemoryEndpointRegistry`, JPA backends, etc. — may fire `EndpointRegistered` or equivalent events after a successful write.
