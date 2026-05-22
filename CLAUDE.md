# CLAUDE.md — casehub/garden

## What This Repo Is

The CaseHub developer knowledge garden. Canonical home for all CaseHub protocols and
platform conventions. Always on `main` — no feature branches, no work branches.

This separation from `casehub/parent` exists deliberately: protocol commits must always
land on main, independent of whatever work branch parent is on. Never create a branch
in this repo.

## Project Type

**Type:** custom

## Protocol Store

All CaseHub protocols live here. Two audiences:

| Index | Audience |
|---|---|
| `docs/protocols/casehub/FOUNDATION-INDEX.md` | Developers building casehub itself (engine, ledger, work, qhorus, platform, connectors) |
| `docs/protocols/casehub/HARNESS-INDEX.md` | Developers building apps on casehub (devtown, aml, clinical, QuarkMind) |
| `docs/protocols/universal/INDEX.md` | Universal Java/Quarkus conventions applicable across all casehub repos |

## Routing

All protocol commits go here. Peer repo sessions (engine, platform, work, etc.) write
protocols to this repo via the path in their CLAUDE.md routing table.

| Artifact | Destination | Notes |
|---|---|---|
| protocols | here | `docs/protocols/casehub/` or `docs/protocols/universal/` based on scope |

## Git Discipline

**Always on main.** No branching. Protocol commits are infrastructure — they go directly
to main so every concurrent session sees them immediately.

```bash
git -C /Users/mdproctor/claude/casehub/garden pull --rebase origin main
git -C /Users/mdproctor/claude/casehub/garden add docs/protocols/...
git -C /Users/mdproctor/claude/casehub/garden commit -m "protocol(...): ..."
git -C /Users/mdproctor/claude/casehub/garden push
```

Always pull before committing — multiple sessions may write protocols concurrently.

## Protocol Skill

The `protocol` skill resolves this repo via the `protocols` routing row in each peer
repo's CLAUDE.md. When invoked from any casehub peer session, it commits here, not
to the peer repo.

## Future Scope

This garden will grow to include:
- casehub-specific knowledge garden entries (gotchas, patterns — currently in hortora garden)
- PENDING-MODULE-UPDATES.md tracks protocols planned but not yet written

The `docs/protocols/` layout mirrors what was in `casehub/parent` until 2026-05-22,
when the store was moved here to solve the branch isolation problem.
