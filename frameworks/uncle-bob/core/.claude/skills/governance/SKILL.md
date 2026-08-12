---
name: governance
description: >
  Unified governance: repo gates, TDD policy, Architecture Decision Records (ADRs),
  evidence-based claims, and shared contracts. Use when making architecture decisions,
  setting up CI/CD gates, writing ADRs, or establishing team agreements.
allowed-tools: Read, Write, Bash, Grep, Glob
---

# governance - Unified Governance

## TDD Policy (Mandatory)

Every feature follows RED → GREEN → REFACTOR:

1. **RED**: write a failing test that describes the desired behavior
2. **GREEN**: write the minimum code to make it pass
3. **REFACTOR**: clean up without breaking the test
4. Target: **≥ 80% coverage** on business logic

```bash
# Verify coverage before considering done
vitest --coverage
```

## Architecture Decision Records (ADRs)

Create an ADR for any decision that:
- Affects the whole team or codebase architecture
- Is hard or costly to reverse
- Involves trade-offs between competing approaches

```markdown
<!-- docs/adr/0001-use-postgresql.md -->
# ADR-0001: Use PostgreSQL as primary database

## Status
Accepted

## Context
We need a relational database. Options considered: PostgreSQL, MySQL, SQLite.

## Decision
Use PostgreSQL.

## Consequences
- Positive: strong JSON support, mature ecosystem, excellent tooling
- Negative: requires managed hosting (added infra cost)
```

## Repo Gates (CI Requirements)

These checks must pass before any merge:

| Gate            | Tool                        | Threshold     |
|-----------------|-----------------------------|---------------|
| Type check      | `tsc --noEmit`              | Zero errors   |
| Lint            | `biome` / `eslint`          | Zero warnings |
| Tests           | `vitest`                    | All pass      |
| Coverage        | coverage report             | ≥ 80%         |
| Commit format   | `commitlint`                | All headers parse (ticket-prefix config from git-workflow.md) |
| Secret scan     | `truffleHog` / `gitleaks`   | Zero hits     |

## Shared Contracts

Define interfaces before implementation:

```typescript
// shared/types/user.ts - agreed before backend/frontend diverge
export interface User {
  id: UserId;
  email: string;
  createdAt: Date;
}

export interface CreateUserInput {
  name: string;
  email: string;
}
```

## Evidence-Based Claims

Don't assert performance or behavior without data:

- ❌ "This is faster" → ✅ "Benchmark shows 40% reduction in p95 latency (see bench/results.json)"
- ❌ "This is more secure" → ✅ "Eliminates SQL injection via parameterized queries (OWASP A03)"

## References

- See [references/adr-template.md](references/adr-template.md) for full ADR template
- See [references/security-gates.md](references/security-gates.md) for security checklist
