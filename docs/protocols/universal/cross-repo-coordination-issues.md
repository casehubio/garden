---
id: PP-20260525-5b1efa
title: "Multi-repo coordination issues belong in the platform root repo only when simultaneous execution is required"
type: rule
scope: universal
applies_to: "Any issue requiring changes spanning multiple repos in a multi-repo platform"
severity: guidance
refs: []
violation_hint: "Filing a new-SPI implementation issue in the platform root repo because 'multiple repos will consume it' — the implementation work is in one repo; file there. Only use the root repo when the work across repos must execute simultaneously and a blocker/blocked-by chain between repo issues is insufficient."
created: 2026-05-25
---

In a multi-repo platform, the root/coordination repo is the inbox for work that requires simultaneous execution across two or more repos — where a blocker/blocked-by chain between individual repo issues is not sufficient because everything must land together. Artifact renames that must propagate atomically, BOM/manifest updates affecting all consumers, CI pipeline changes, and platform-wide sweeps are coordination-repo issues. A new SPI in a module repo that consuming repos will later implement is NOT a coordination-repo issue — the implementation work is in one place, and consumers follow sequentially. File in the coordination repo only when the work itself cannot be sequenced across repos.
