---
id: PP-20260531-300481
title: "Trust policy base fields come from CapabilityRegistry; engine-specific fields come from YAML"
type: rule
scope: application
applies_to: "devtown TrustRoutingPolicyProvider; any future harness implementing TrustRoutingPolicyProvider"
severity: important
refs:
  - devtown/docs/specs/2026-05-30-layer6-trust-routing-design.md
  - casehub-parent/docs/protocols/casehub/trust-maturity-model.md
violation_hint: "threshold, minimumObservations, or borderlineMargin declared in both CapabilityRegistry.POLICIES and in the trust-routing YAML — values will silently diverge over time"
created: 2026-05-31
---

`TrustRoutingPolicyProvider` implementations MUST read `threshold`, `minimumObservations`, and `borderlineMargin` from `CapabilityRegistry.policy()` — these are the three base fields present in `RoutingPolicy`. They must not be duplicated in YAML config or hardcoded in the provider. `DevtownCapabilityRegistry.POLICIES` is the single source of truth for these values. YAML carries only fields absent from `RoutingPolicy` — `blendFactor` and `qualityFloors` — which are engine-specific tuning parameters with no domain representation. Splitting this way prevents silent data divergence between the domain registry and the engine config layer.
