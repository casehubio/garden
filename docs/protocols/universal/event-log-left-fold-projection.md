---
id: PP-20260602-b748c9
title: "Project event-log history into derived views using a left-fold over typed events"
type: rule
scope: universal
applies_to: "Any component building a summary, digest, or read-model from an append-only typed event or message log"
severity: important
refs: []
violation_hint: "Ad-hoc parsing of event content to build a summary view, or delegating to an LLM for summarization — produces non-deterministic output and couples the summary shape to message internals rather than to typed event structure."
created: 2026-06-02
---

When projecting an append-only typed event log into a derived view (summary, digest, changelog, read-model), implement as a pure left-fold: (currentState, event) → newState. Each event type maps to a stateless handler that updates the materialized state; the final state is rendered to the target format by a separate, deterministic renderer. This ensures identical inputs always produce identical outputs, makes the projection independently testable without the rendering layer, and cleanly separates event consumption from view generation. The fold can be replayed from any cursor position for incremental updates. Do not parse free-form content or use an LLM to synthesize the view — the type system of the event log is the schema; the fold is the transformation.
