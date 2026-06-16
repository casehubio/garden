---
id: PP-20260604-a7ad99
title: "allowedTypes and deniedTypes are architectural invariants — only set when a hard constraint must be enforced across all scenarios"
type: rule
scope: platform
applies_to: "Any CaseChannelLayout implementation or any code creating Qhorus channels"
severity: important
refs:
  - claudony#142
created: 2026-06-04
---

Type enforcement on Qhorus channels is discriminated by normative weight.

**COMMAND and QUERY — hard-enforced.**
Both types call `commitmentService.open()`. Advisory dispatch on the wrong channel followed by
LLM correction creates orphan OPEN Commitments — stalled permanently with no mechanism to
distinguish them from genuine governance failures. Hard enforcement is normatively correct for
these types: a directive that cannot be honoured on a channel where no agent responds is not a
valid speech act.

**All other types — advisory.**
STATUS, EVENT, DONE, FAILURE, DECLINE, RESPONSE, HANDOFF do not call `commitmentService.open()`.
Advisory dispatch for these types produces an accurate audit entry (WARN log + ledger entry +
`DispatchResult.advisories()`) without orphan risk. Hard enforcement for non-obligation-creating
types erases constraint violations from the ledger; advisory enforcement makes them visible.

The `observe`/`oversight`/`work` normative layout examples remain valid:
- `observe` (`allowedTypes=EVENT`): COMMAND and QUERY on observe are still hard-blocked.
  STATUS and other non-obligation-creating types on EVENT-only channels trigger advisories.
- `oversight` (`deniedTypes=EVENT`): EVENT on oversight triggers an advisory (EVENTs are
  already excluded from `pollAfter` defaults; the advisory record is more informative than
  a hard block).
- `work` (open): no constraints; no enforcement.

`allowedTypes` and `deniedTypes` remain valid channel configuration. They declare intent and
drive enforcement at the appropriate strength for each type category.
