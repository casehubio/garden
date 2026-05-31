---
id: PP-20260531-0cee0c
title: "Every GitHub issue must carry scale and complexity labels at creation"
type: rule
scope: platform
applies_to: "All casehub GitHub repositories"
severity: important
refs:
  - docs/protocols/casehub/FOUNDATION-INDEX.md
violation_hint: "An issue with no scale: * or complexity: * label — creates triage debt and forces re-analysis at action time"
created: 2026-05-31
---

Every issue filed against a casehub repo must have exactly one `scale: *` label (XS / S / M / L / XL) and one `complexity: *` label (Low / Med / High) applied at creation. Infer both from the issue title and body — do not ask the reporter. Unlabelled issues cannot be prioritised or scheduled without first re-reading them in full; at scale this accumulates into a triage tax that makes the issue backlog unactionable. The issue-workflow skill enforces this automatically via Phase 1 and Phase 2 prompts; all repos carry the eight label definitions as a prerequisite to issue tracking setup.
