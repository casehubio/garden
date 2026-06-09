---
id: PP-20260609-a443ad
title: "All debate channel entryType values must use EntryType.name() — never lowercase strings"
type: rule
scope: application
applies_to: "Any class encoding or decoding DraftHouse debate channel messages: DebateMcpTools, ChannelAgentDispatcher handler subclasses, DebateChannelProjection"
severity: critical
refs:
  - docs/superpowers/specs/2026-06-09-review-session-continuity-design.md
violation_hint: "entryType=raise, entryType=agree, or entryType=flag-human in message content — lowercase strings that fail EntryType.valueOf() silently discarding messages"
created: 2026-06-09
---

All structured debate channel messages encode their entry type using `EntryType.name()` as the wire format — always uppercase enum names (`RAISE`, `AGREE`, `FLAG_HUMAN`, `SUB_TASK_REQUEST`, etc.). `DebateChannelProjection.apply()` decodes via `EntryType.valueOf(entryTypeStr)` and discards unknown values with a WARNING log. Mixed-case encoding (old lowercase, new uppercase) causes silently discarded messages with no error — the projection folds the message as state-unchanged. Encoding with the Java enum name guarantees the decode always succeeds for known types. Do not hand-code lowercase entry type strings anywhere; always use `EntryType.name()` or the enum constant's `name()` method when building META headers.
