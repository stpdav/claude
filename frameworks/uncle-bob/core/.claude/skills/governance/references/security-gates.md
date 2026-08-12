# Security Gates Reference

## CI Security Gates

These run on every PR - a failure blocks the merge:

| Gate              | Tool                          | Threshold            |
|-------------------|-------------------------------|----------------------|
| Secret scan       | `gitleaks` / `truffleHog`     | Zero verified hits   |
| Dependency audit  | `pnpm audit`                  | Zero high / critical |
| Lint (security)   | `biome` / `eslint`            | Zero errors          |

```yaml
# .github/workflows/ci.yml - security steps
      - name: Secret scan
        run: trufflehog filesystem . --only-verified

      - name: Dependency audit
        run: pnpm audit --audit-level high
```

## Per-PR Security Checklist

- [ ] No hardcoded secrets, API keys, or credentials - environment variables only
- [ ] All external input validated at the boundary (schema validation, e.g. zod)
- [ ] Parameterized queries only - no string-concatenated SQL (OWASP A03)
- [ ] Auth/permission checks on every protected route, Route Handler, and Server Action
- [ ] No `eval()` or dynamic code execution reachable from user input
- [ ] Error responses don't leak internals - no stack traces, SQL, or file paths to clients
- [ ] `.env` files ignored by git; `.env.example` kept up to date

## Dependency Hygiene

- Lockfile (`pnpm-lock.yaml`) always committed - installs are reproducible or they're broken
- New dependencies justified in the PR description: what it does, why not stdlib
- Prefer zero-dependency solutions for trivial logic (left-pad rule)

## Incident Rule

A leaked secret is rotated immediately - removing it from git history is cleanup, not remediation.
