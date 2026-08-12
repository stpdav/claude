# State Management Reference

## Decision Ladder

Start at the top; move down only when the current level hurts:

1. **Local `useState`** - UI state owned by one component
2. **Lifted state** - shared by siblings → lift to the closest common parent
3. **Context** - read by many components across a subtree (theme, session, locale)
4. **State library** (Zustand / Jotai) - frequent cross-tree updates where context re-renders hurt
5. **Server state** (TanStack Query / SWR) - anything fetched from an API - never mirror it into `useState`

## Derive, Don't Sync

```tsx
// ❌ Synced copy - drifts and re-renders twice
const [fullName, setFullName] = useState("");
useEffect(() => setFullName(`${first} ${last}`), [first, last]);

// ✅ Derive during render - always correct
const fullName = `${first} ${last}`;
```

## Context Without Re-Render Pain

Split value and updater into separate contexts - consumers of the setter don't re-render on value changes:

```tsx
const CountContext = createContext<number>(0);
const CountDispatchContext = createContext<(n: number) => void>(() => {});

export function CountProvider({ children }: { children: ReactNode }) {
  const [count, setCount] = useState(0);
  return (
    <CountContext.Provider value={count}>
      <CountDispatchContext.Provider value={setCount}>
        {children}
      </CountDispatchContext.Provider>
    </CountContext.Provider>
  );
}

export const useCount = () => useContext(CountContext);
export const useSetCount = () => useContext(CountDispatchContext);
```

Memoize object context values - a fresh `{}` every render re-renders every consumer:

```tsx
const value = useMemo(() => ({ user, logout }), [user, logout]);
```

## Reducers for Multi-Field Transitions

When several state fields change together, one `useReducer` beats N `useState` calls:

```tsx
type Action =
  | { type: "loading" }
  | { type: "success"; data: User }
  | { type: "error"; error: Error };
```

## Anti-Patterns

- **Server data in `useState`**: no caching, no revalidation, no dedup - use a server-state library
- **Context as global store**: high-frequency updates through context re-render whole subtrees
- **Prop drilling past 2 levels**: compose (`children`) or use context
- **State for derivable values**: if it can be computed from existing state/props, compute it
