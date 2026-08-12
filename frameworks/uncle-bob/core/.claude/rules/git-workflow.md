# Git Workflow Rules

## Commit Messages (Conventional Commits)

Format: `[TICKET-ID] type(scope): description` - prefix omitted only for unticketed work

| Type     | When to use                              |
|----------|------------------------------------------|
| feat     | New feature                              |
| fix      | Bug fix                                  |
| refactor | Code change that neither fixes nor adds  |
| test     | Adding or updating tests                 |
| docs     | Documentation only                       |
| chore    | Build, tooling, or dependency updates    |

Examples:
```
[ABC-123] feat(auth): add JWT refresh token support
[ABC-456] fix(user): handle null email in profile update
test(cart): add edge cases for empty cart
```

## Branch Naming

`type/TICKET-ID-short-description` - e.g. `feat/ABC-123-user-auth`, `fix/ABC-456-null-email`

Unticketed work (rare): `type/short-description`

## Ticket References

Ticketed work carries its ticket ID in three places. Works with any tracker whose IDs
match `[A-Z][A-Z0-9]*-[0-9]+` (YouTrack, Jira, Linear, ...):

| Where         | Format                                 | Example                                  |
|---------------|----------------------------------------|------------------------------------------|
| Branch name   | `type/TICKET-ID-short-description`     | `refactor/ABC-123-split-into-layers`     |
| Commit header | `[TICKET-ID] type(scope): description` | `[ABC-123] refactor(thing): split layers`|
| PR title      | `[TICKET-ID] type(scope): description` | `[ABC-123] refactor(thing): split layers`|

- One convention everywhere - commits, PR titles, and squash-merges all carry the same prefix
- `/create-pr` derives the ID from the branch name automatically
- The prefix requires the commitlint config below - vanilla Conventional Commits parsers reject it

## Commitlint Config (required)

Every repo ships this so commitlint, semantic-release, and changelog generators accept the prefix:

```js
// commitlint.config.js
export default {
  extends: ["@commitlint/config-conventional"],
  parserPreset: {
    parserOpts: {
      // Tolerate an optional [TICKET-ID] prefix before the conventional header
      headerPattern: /^(?:\[[A-Z][A-Z0-9]*-\d+\]\s+)?(\w+)(?:\(([^)]*)\))?!?: (.+)$/,
      headerCorrespondence: ["type", "scope", "subject"],
    },
  },
};
```

semantic-release and conventional-changelog take the same `parserOpts` - reuse the pattern there.

## Before Every Commit

Run in order:
1. Linter (`biome check` / `eslint`)
2. Type checker (`tsc --noEmit`)
3. Tests (`vitest run`)

Never commit if any of these fail.

## Never Commit

- Secrets, API keys, or credentials
- `console.log` debug statements
- Commented-out code blocks
- `.env` files (use `.env.example` instead)
