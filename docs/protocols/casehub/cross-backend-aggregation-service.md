---
id: PP-20260601-c60b28
title: "Cross-backend aggregation services must isolate failures per source and track partial results with a sources field"
type: rule
scope: platform
applies_to: "Any casehub module or application that assembles a unified view by querying multiple independent backends (different persistence units, SPIs, or in-memory stores)"
severity: important
refs:
  - docs/protocols/casehub/coordinator-no-transactional-multi-datasource.md
  - docs/protocols/casehub/multi-datasource-ledger-work-qhorus.md
violation_hint: "An aggregation endpoint returns 500 when one backend is unavailable, or omits a sources field so callers cannot detect partial results programmatically"
created: 2026-06-01
---

When an endpoint aggregates data from N independent backends (trust scores from ledger, active WorkItems from work, open Commitments from qhorus, running cases from engine), each backend must be queried independently with its own failure domain. Wrap each source in a try-catch; on exception, return empty/null for that source and exclude its name from the `sources` array — never propagate as a 500. The response is always 200 with a `sources` field listing which backends actually responded, so callers can detect partial results programmatically. Never open a shared JTA transaction across backends. When a result from backend A requires a follow-up lookup in backend B per row (e.g. Commitment → Channel name), use a JOIN projection query rather than N+1 per-row lookups. Document — and do not attempt to enforce at runtime — that the key used to query across backends must be the same string in every system; silent empty results from one source indicate identity misalignment, not an error.
