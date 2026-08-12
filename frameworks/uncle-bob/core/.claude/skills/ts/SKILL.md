---
name: ts
description: >
  TypeScript and JavaScript best practices (latest stable). Use when writing,
  reviewing, or debugging .ts, .tsx, .js, or .jsx files. Covers strict typing,
  module resolution, tsconfig architecture, and common patterns.
  Auto-activates on TypeScript and JavaScript files.
allowed-tools: Read, Bash, Grep, Glob
---

# ts - TypeScript Best Practices

## tsconfig (Strict Mode Required)

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "moduleResolution": "bundler",
    "module": "ESNext",
    "target": "ESNext"
  }
}
```

## Typing Rules

```typescript
// Never use `any` - use `unknown` + type guards
function parse(input: unknown): string {
  if (typeof input !== "string") throw new TypeError("Expected string");
  return input;
}

// Prefer type over interface for unions/intersections
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string };

// Use branded types for IDs
type UserId = string & { readonly __brand: "UserId" };
const toUserId = (id: string): UserId => id as UserId;

// Prefer `satisfies` for config objects
const config = {
  host: "localhost",
  port: 3000,
} satisfies ServerConfig;
```

## Module Resolution

```typescript
// Always use explicit extensions in ESM
import { getUser } from "./user.js";   // ✅
import { getUser } from "./user";      // ❌ in ESM

// Barrel files - use sparingly, only at package boundaries
// index.ts should export public API only
```

## Error Handling

```typescript
// Use typed errors - never throw raw strings
class ValidationError extends Error {
  constructor(
    message: string,
    public readonly field: string
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

// Prefer Result pattern for expected failures
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

## Tooling

```bash
# Type check
tsc --noEmit

# Lint + format
biome check . --write       # or: eslint . && prettier --write .
```

## Rules

- TypeScript only - never create `.js` / `.jsx` source files; plain JS is allowed only for configs the tool cannot load as TS
- `strict: true` is mandatory - never disable
- No `any` - use `unknown` with guards
- All exported functions are typed (no implicit returns)
- Run `tsc --noEmit` before finishing
- Use `pnpm` for package management and running scripts

## References

- See [references/async-patterns.md](references/async-patterns.md) for async/await
- See [references/generics.md](references/generics.md) for advanced generics
