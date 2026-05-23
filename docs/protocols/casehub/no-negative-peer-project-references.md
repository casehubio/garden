---
id: PP-20260523-6ae1e1
title: "Do not reference peer projects negatively in external-facing content"
type: rule
scope: platform
applies_to: "all external-facing content — website, landing pages, READMEs, blog posts, marketing materials"
severity: important
refs:
  - casehubio.github.io/CLAUDE.md
violation_hint: "Content that says 'no Camel', 'unlike X', 'without needing Y' where Y is a named open source project maintained by colleagues or the broader community"
created: 2026-05-23
---

CaseHub's implementation choices should be described in terms of what they do, not what they avoid. Named peer projects — Apache Camel, Quarkus extensions, Spring components, or any other community project — must not appear as negative comparisons in external content, even when the distinction is technically accurate. Describe the approach positively ("pure HttpClient", "no external messaging framework dependencies") rather than by exclusion ("no Camel", "not Spring"). This applies regardless of whether the comparison is flattering to CaseHub — the community relationships matter more than the positioning.
