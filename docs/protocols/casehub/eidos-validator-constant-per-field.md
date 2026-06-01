---
id: PP-20260601-347bba
title: "Each AgentDescriptorValidator field must have its own named constant"
type: rule
scope: repo
applies_to: "AgentDescriptorValidator in casehub-eidos-api"
severity: guidance
refs: []
violation_hint: "MAX_PROVIDER or MAX_JURISDICTION used as the bound for a field they don't describe (e.g. modelFamily validated against MAX_PROVIDER)"
created: 2026-06-01
---

Every optional or required field validated in `AgentDescriptorValidator` must have its own
named constant — even when the numeric bound is identical to another field's constant. A
constant named `MAX_PROVIDER` bounding `modelFamily` is a violation: the name misleads
readers into thinking the fields share a semantic bound, not just a numeric coincidence.
When two compliance-text fields share a bound, both get their own constant and a
cross-reference comment (e.g. `MAX_DATA_HANDLING_POLICY = 1000; // compliance-text; same
bound as MAX_JURISDICTION`). Discovered and fixed in eidos#24.
