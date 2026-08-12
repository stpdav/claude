---
name: quality
description: >
  Cross-language code quality standards. Use when writing, reviewing, or refactoring
  any code. Covers design patterns, formatting, documentation, structure, naming,
  and clean code principles. ALWAYS loaded - applies to every task.
allowed-tools: Read, Bash, Grep, Glob
---

# quality - Cross-Language Code Quality

Core clean-code principles by Robert C. Martin, adapted for AI-assisted development.

## Non-Negotiable Rules

- **Single Responsibility**: one function/class = one reason to change
- **DRY**: never duplicate logic - extract, reuse, reference
- **Explicit over implicit**: readable code beats clever code
- **Names describe intent**: `getUserById` not `getU`, `isActive` not `flag`
- **No magic numbers**: use named constants
- **Functions ≤ 20 lines**: if longer, extract
- **Files ≤ 300 lines**: if longer, split
- **No dead code**: remove commented-out blocks and unused variables
- **No side effects**: pure functions where possible

## Naming Conventions

| Construct   | Style               | Example                      |
|-------------|---------------------|------------------------------|
| Variables   | camelCase           | `userId`                     |
| Functions   | verb + noun         | `fetchUser`, `validateEmail` |
| Classes     | PascalCase          | `UserRepository`             |
| Constants   | UPPER_SNAKE_CASE    | `MAX_RETRY_COUNT`            |
| Files       | kebab-case          | `user-repository.ts`         |
| Booleans    | is/has/can prefix   | `isReady`, `hasPermission`   |

## Documentation Rules

- Every public function/method/class gets a JSDoc comment
- Document the **why**, not the what (code shows the what)
- Keep comments up to date - stale comments are worse than none

## Error Handling

- Never swallow errors silently
- Use typed errors / custom error classes
- Log with context: what failed, where, what input caused it
- Fail fast at boundaries; handle gracefully at the top

## Refactoring Checklist

Before finishing any edit, verify:
- [ ] No duplicated logic
- [ ] All names clearly describe intent
- [ ] No function does more than one thing
- [ ] No magic numbers or strings
- [ ] Error paths are handled
- [ ] Public surface is documented

## References

- See [references/patterns.md](references/patterns.md) for design pattern guidance
- See [references/formatting.md](references/formatting.md) for formatting rules per language
