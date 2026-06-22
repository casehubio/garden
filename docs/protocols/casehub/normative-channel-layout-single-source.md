---
id: PP-20260622-normative-layout
title: "Use NormativeChannelLayout from casehub-engine-api — do not define private layout maps"
type: rule
scope: platform
applies_to: "Any casehub module that creates Qhorus channels per agent case — claudony, openclaw, and any future agent implementation"
severity: important
refs:
  - casehubio/parent#93
violation_hint: >
  A private ChannelSpec record or Map<String, ChannelSpec> defined inside a CaseChannelProvider
  or ReactiveCaseChannelProvider — bypasses NormativeChannelLayoutTest value assertions in
  engine-api and re-introduces the duplication parent#93 eliminated.
created: 2026-06-22
---

# Convention: Use NormativeChannelLayout from casehub-engine-api

**Applies to:** Any casehub module that creates Qhorus channels per agent case  
**Severity:** Important — private layout maps drift silently and bypass engine-api test guards

## The Rule

`NormativeChannelLayout` in `io.casehub.api.spi.mesh` (`casehub-engine-api`) is the single
source of truth for the three normative agent mesh channels (work / observe / oversight) and
their `MessageType` constraints.

Use `CaseChannelLayout.named("normative")` or construct `new NormativeChannelLayout()` directly.
For the lightweight 2-channel variant (no oversight gate), use `CaseChannelLayout.named("simple")`
or `new SimpleLayout()`.

**Do not define private `ChannelSpec` records or `LAYOUT` maps inside provider classes.**

## Why

Before parent#93, claudony owned `NormativeChannelLayout` and openclaw had its own
`OpenClawNormativeLayout` — an exact duplicate. Both had to be kept in sync manually.
When the normative layout changes (e.g. a new channel type, a constraint update), there is
now exactly one place to change: `NormativeChannelLayout` in engine-api. `NormativeChannelLayoutTest`
in engine-api guards the exact `allowedTypes`/`deniedTypes` values per protocol.

A private layout in a provider bypasses this guard entirely — the violation won't be caught
until a runtime type-enforcement error surfaces, or until the channels diverge silently.

## Where This Applies

- `claudony-casehub` — `ClaudonyReactiveCaseChannelProvider` (uses `CaseChannelLayout.named(config.channelLayout())`)
- `openclaw-casehub` — `OpenClawCaseChannelProvider` and `ReactiveOpenClawCaseChannelProvider`
- Any future agent implementation that creates Qhorus channels per case
