# Code Quality Rules

Always apply these rules on every file touched.

## Structure
- One function = one responsibility
- Functions ≤ 20 lines. If longer, extract helpers.
- Files ≤ 300 lines. If longer, split by responsibility.
- No dead code: remove commented-out blocks and unused variables.

## Naming
- Variables and functions: describe intent (`getUserById`, not `getU`)
- Booleans: `is`, `has`, or `can` prefix (`isActive`, `hasPermission`)
- Constants: `UPPER_SNAKE_CASE`
- No abbreviations unless universally known (`id`, `url`, `api` are fine)

## Documentation
- Every exported function gets a JSDoc comment
- Comment the *why*, not the *what* - code shows the what
- Keep comments up to date - stale comments are worse than none

## Error Handling
- Never swallow errors silently
- Log with context: what failed, where, what input caused it
- Use typed/custom error classes for domain errors

## No Magic Values
- No hardcoded numbers or strings - use named constants
- No hardcoded credentials, API keys, or secrets - use environment variables
