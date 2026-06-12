---
id: PP-20260612-042941
title: "Reserve SPI property key constants only for keys whose values cross module boundaries"
type: rule
scope: platform
applies_to: "Any SPI that exposes a Map<String,String> properties bag (EndpointDescriptor, future SPIs)"
severity: important
refs:
  - docs/protocols/casehub/platform-api-scope.md
  - docs/protocols/universal/module-tier-structure.md
violation_hint: "Adding a constant to EndpointPropertyKeys (or equivalent) for a key that is written and read within the same module"
created: 2026-06-12
---

# Protocol: SPI Property Key Constants — Cross-Module Only

**Applies to:** Any SPI with a `Map<String,String>` properties bag  
**Severity:** Important — premature constants inflate shared API surface and create coupling between modules that need not be coupled

When a CasHub platform SPI exposes a `Map<String,String>` properties bag (e.g.
`EndpointDescriptor.properties()`), a companion constants class (e.g. `EndpointPropertyKeys`)
must only reserve keys whose values are **read by a different module than the one that wrote them**.
Deployment-local keys — bootstrap servers, TLS config, route IDs, API keys, internal correlation
IDs — are module-specific and must remain as module-defined string literals, not platform constants.

The test: *"Could a second module, independently, register an endpoint (or any SPI value) using
key X and expect a first module to read X back?"* If yes, reserve the key as a constant. If no,
keep it local. Reference: `EndpointPropertyKeys.URL` (HTTP workers and Camel workers both read it)
and `EndpointPropertyKeys.TOPIC` (Kafka producers and consumers must agree on the topic key)
are the two initial reserved keys — all others are deployment-local until a concrete cross-module
consumer demands them.
