---
name: react
description: >
  React (latest stable) best practices. Use when writing or reviewing .tsx/.jsx files,
  custom hooks, or React component logic. Covers functional components, hooks, reactivity,
  performance, and strict TypeScript integration.
  Auto-activates on .tsx and .jsx files.
allowed-tools: Read, Bash, Grep, Glob
---

# react - React Best Practices

## Component Structure

Always use functional components with TypeScript. Never use class components.

```tsx
// components/UserCard.tsx

interface UserCardProps {
  userId: string;
  label?: string;
  onSelect: (userId: string) => void;
}

export function UserCard({ userId, label, onSelect }: UserCardProps) {
  const { user, isLoading } = useUser(userId);

  if (isLoading) return <Spinner />;
  if (!user) return null;

  return (
    <div onClick={() => onSelect(userId)}>
      <span>{label ?? user.name}</span>
    </div>
  );
}
```

- One component per file, filename matches component name (`UserCard.tsx`)
- Export named, not default - easier to refactor and search
- Props interface defined above the component, never inline

## Custom Hooks

Extract stateful logic into custom hooks. Never duplicate state management across components.

```typescript
// hooks/useUser.ts
import { useState, useEffect } from "react";

export function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      setIsLoading(true);
      setError(null);
      try {
        const data = await api.getUser(userId);
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err instanceof Error ? err : new Error(String(err)));
      } finally {
        if (!cancelled) setIsLoading(false);
      }
    }

    fetchUser();

    // Cleanup prevents state update on unmounted component
    return () => { cancelled = true; };
  }, [userId]);

  return { user, isLoading, error };
}
```

Hook naming rules:
- Always prefix with `use` (`useUser`, `useCart`, `useDebounce`)
- Return a plain object, not an array (unless it mimics `useState`)
- Clean up subscriptions and async operations in the `useEffect` return

## Hooks Rules

```tsx
// ❌ Never call hooks conditionally
if (isAdmin) {
  const data = useFetch("/admin");  // breaks Rules of Hooks
}

// ✅ Condition goes inside the hook or component body
const data = useFetch(isAdmin ? "/admin" : null);
```

- `useState` - local UI state only. Shared state belongs in context or a state library.
- `useEffect` - for synchronising with external systems (API, DOM, timers). Not for derived state.
- `useMemo` - memoize expensive computations, not simple values.
- `useCallback` - memoize callbacks passed to child components that are wrapped in `React.memo`.
- Never lie in dependency arrays - include every value used inside the effect.

## Props & Types

```typescript
// ✅ Explicit prop types
interface ButtonProps {
  label: string;
  isDisabled?: boolean;
  onClick: () => void;
  variant: "primary" | "secondary" | "danger";
}

// ❌ Never use any or object
interface BadProps {
  data: any;
  handler: object;
}
```

- Booleans default to `false` - declare `isDisabled?: boolean`, not `isDisabled: boolean`
- Callback props named `onXxx` (`onClick`, `onChange`, `onSelect`)
- Avoid prop drilling beyond 2 levels - use context or composition

## Performance

```tsx
// Memoize a component that receives stable props from an expensive parent
const UserCard = React.memo(function UserCard({ userId, onSelect }: UserCardProps) {
  return <div onClick={() => onSelect(userId)} />;
});

// Stable callback reference so memo is not broken by parent re-renders
const handleSelect = useCallback((userId: string) => {
  setSelectedId(userId);
}, []);  // stable - no dependencies that change

// Memoize a costly derivation
const sortedUsers = useMemo(
  () => [...users].sort((a, b) => a.name.localeCompare(b.name)),
  [users]
);
```

Do not wrap everything in `memo`/`useMemo`/`useCallback` by default - profile first.

## Error Boundaries

Wrap feature subtrees in error boundaries to prevent full-page crashes:

```tsx
// components/ErrorBoundary.tsx
import { Component, type ReactNode } from "react";

interface Props { children: ReactNode; fallback: ReactNode; }
interface State { hasError: boolean; }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  render() {
    return this.state.hasError ? this.props.fallback : this.props.children;
  }
}
```

## Tooling Workflow

```bash
# Install
pnpm install

# Dev server
pnpm run dev

# Type check
pnpm exec tsc --noEmit

# Lint
pnpm exec biome check --write .

# Test
pnpm exec vitest run
pnpm exec vitest --coverage   # enforce ≥ 80% on business logic
```

## Testing (React Testing Library)

```tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import { UserCard } from "./UserCard";

describe("UserCard", () => {
  it("calls onSelect with userId when clicked", () => {
    const onSelect = vi.fn();
    render(<UserCard userId="1" onSelect={onSelect} />);
    fireEvent.click(screen.getByRole("button"));
    expect(onSelect).toHaveBeenCalledWith("1");
  });

  it("renders label when provided", () => {
    render(<UserCard userId="1" label="Alice" onSelect={vi.fn()} />);
    expect(screen.getByText("Alice")).toBeInTheDocument();
  });
});
```

- Test **behaviour**, not implementation details (no `instance`, no internal state access)
- Query by accessible roles first: `getByRole`, `getByLabelText`, `getByText`
- Avoid `getByTestId` - it couples tests to implementation

## Rules

- Functional components only - no class components
- Named exports only - no default exports for components
- All props explicitly typed with a TypeScript interface
- Custom hooks for all stateful logic shared across components
- Never ignore `useEffect` dependencies - fix the root cause instead
- Run `tsc --noEmit` and `biome check` before finishing any task
- ≥ 80% test coverage on business logic

## References

- See [references/state.md](references/state.md) for context and global state patterns
- See [references/testing.md](references/testing.md) for advanced React Testing Library patterns
