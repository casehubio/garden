# Failure Cascade Pattern

**Scope:** Application-tier harnesses using CaseHub engine for agent-dispatched work.
**Applies when:** A CasePlanModel dispatches work to agents and must handle DECLINED, FAILED, or EXPIRED outcomes.

---

## Pattern

When an agent cannot complete dispatched work, the system responds through a four-tier cascade. The engine owns the reroute loop (Tiers 1-2); the application owns domain-specific responses (Tiers 3-4).

### Tier 1-2: Engine-owned reroute (OutcomePolicy)

Configure `outcomePolicy` on each binding:

```yaml
outcomePolicy:
  onDecline: REROUTE
  onFailure: REROUTE
  onExpired: REROUTE
  maxRerouteAttempts: 2
```

The engine handles: writing structured failure state to the blackboard, adding the agent to `excludedAgents`, incrementing `attempts`, re-dispatching with agent exclusion. When `maxRerouteAttempts` is reached, the engine writes `status: "REROUTES_EXHAUSTED"`.

### Tier 3: Application-owned scope reduction

Fires when reroutes are exhausted and the capability supports scope reduction. Uses `contextWrite` for pre-dispatch state marking and `inputSchemaOverride` for narrowed input:

```yaml
- name: {cap}-reduced-scope
  when: '.{cap}.status == "REROUTES_EXHAUSTED" and .{cap}.reducedScope == null'
  contextWrite:
    {cap}:
      status: PENDING
      reducedScope: true
      excludedAgents: []
  capability: {cap}
  inputSchemaOverride: "{ flaggedFiles: .codeAnalysis.flaggedFiles }"
  outcomePolicy: { onDecline: REROUTE, onFailure: REROUTE, onExpired: REROUTE, maxRerouteAttempts: 2 }
  conflictResolverStrategy: DEEP_MERGE
```

### Tier 4: Application-owned human escalation

Fires when all automated tiers are exhausted:

```yaml
- name: {cap}-human-escalation
  when: '.{cap}.status == "REROUTES_EXHAUSTED" and (.{cap}.reducedScope == true or NOT scopeReductionAllowed)'
  conflictResolverStrategy: DEEP_MERGE
  humanTask:
    outcomes: [APPROVED, REJECTED, BLOCKED]
```

---

## Outcome Distinctions

| Outcome | Meaning | Routing | Trust dimension |
|---------|---------|---------|----------------|
| DECLINED | Capability boundary — agent is healthy | Reroute | scope-calibration (positive) |
| FAILED | Execution error — agent tried | Reroute | review-thoroughness (negative) |
| EXPIRED | Commitment deadline passed — agent silent | Reroute + investigation | responsiveness (negative) |

All three produce the same routing response (REROUTE) but feed different trust scoring dimensions. RetryPolicy (existing) handles infrastructure exceptions (worker threw) — a separate mechanism from OutcomePolicy.

---

## Blackboard Failure State Schema

The engine writes structured failure state under the capability's blackboard key:

```json
{
  "status": "DECLINED | FAILED | EXPIRED | REROUTES_EXHAUSTED | PENDING | COMPLETED",
  "outcome": "APPROVED | REJECTED | BLOCKED",
  "attempts": 1,
  "history": [{ "agent": "agent-a", "status": "DECLINED", "reason": "...", "timestamp": "..." }],
  "excludedAgents": ["agent-a"],
  "reducedScope": true
}
```

- `status` — engine-managed dispatch state
- `outcome` — only on successful completion (domain semantics)
- `history` — append-only audit trail
- `excludedAgents` — accumulates within a reroute cycle, resets on scope reduction
- `reducedScope` — set by Tier 3 contextWrite

---

## Key Contracts

- **contextWrite uses JSON Merge Patch (RFC 7396):** objects merge recursively; scalars and arrays replace. This ensures `excludedAgents: []` replaces (clears) while `history` is preserved.
- **DEEP_MERGE on all failure-tracking bindings:** both worker output and humanTask output paths must use DEEP_MERGE to prevent success output from overwriting failure tracking state.
- **Failure goals produce COMPLETED, not FAULTED:** FAULTED is for system errors. A review that exhausted all tiers is a legitimate process conclusion with failure outcome metadata.
- **Scope reduction resets excludedAgents:** a DECLINE for full scope doesn't imply inability at narrowed scope.

---

## Engine Dependencies

| Issue | What |
|-------|------|
| engine#502 | Agent exclusion via `excludedAgents` in routing |
| engine#503 | Structured failure state blackboard write |
| engine#504 | OutcomePolicy (REROUTE/FAULT for speech-act outcomes) |
| engine#506 | Failure goals → COMPLETED not FAULTED |
| engine#508 | DEEP_MERGE conflict resolver (worker + humanTask paths) |
| engine#509 | Binding.inputSchemaOverride |
| engine#511 | Binding.contextWrite (pre-dispatch state marking) |
| engine#512 | HumanTaskTarget.outcomes |
| qhorus#281 | CommitmentExpiredEvent CDI event |

---

## FailurePolicy (Application-Level Descriptor)

Each capability declares its Tier 3/4 behavior:

```java
public record FailurePolicy(
    boolean scopeReductionAllowed,
    String reducedInputSchema,
    Duration humanEscalationSla
) {}
```

Reroute parameters (`maxRerouteAttempts`) are on `OutcomePolicy` per binding — engine-owned, not application-owned.
