# ADR Template Reference

Copy into `docs/adr/NNNN-short-title.md`, numbered sequentially (0001, 0002, ...).

```markdown
# ADR-XXXX: [Short title in imperative mood]

## Status
Proposed | Accepted | Superseded by ADR-XXXX

## Context
[What problem are we solving? What constraints exist - technical, team, budget, deadline?]

## Options Considered

### Option A: [Name]
- Pro: ...
- Con: ...

### Option B: [Name]
- Pro: ...
- Con: ...

## Decision
[What we chose and why - reference the evidence, not opinions]

## Consequences
- Positive: ...
- Negative: ...
- Risks: ...

## Date
YYYY-MM-DD
```

## Rules

- One decision per ADR - split compound decisions
- Never edit an accepted ADR - write a new one that supersedes it
- Every option needs at least one honest con - an option with no cons wasn't evaluated
- Claims in Context and Decision follow the evidence standard: benchmark, spec, or measurement
- Link the ADR from the PR that implements it
