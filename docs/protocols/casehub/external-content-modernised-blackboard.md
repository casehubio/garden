---
id: PP-20260523-95ed58
title: "Use 'Modernised Blackboard Architecture' in external-facing content — not 'CMMN'"
type: rule
scope: platform
applies_to: "all external-facing content — website, landing pages, READMEs, blog posts, marketing materials"
severity: guidance
refs:
  - casehubio.github.io/CLAUDE.md
violation_hint: "Content that says 'CMMN semantics', 'CMMN process model', or uses CMMN as a label in any public-facing surface"
created: 2026-05-23
---

CaseHub's orchestration model is informed by but distinct from CMMN (Case Management Model and Notation). External communications must use "Modernised Blackboard Architecture" to describe the coordination model — this accurately represents what CaseHub does (blackboard-driven coordination with adaptive routing) without implying strict CMMN conformance, which the platform does not claim. CMMN may appear in internal design docs and ADRs where precision matters, but never in content a visitor, user, or evaluator reads first.
