# Generics Reference

## When to Reach for a Generic

- A function behaves identically across many types → generic parameter
- Parameter and return types vary together → tie them with one parameter
- Only one call site or one concrete type → don't - use the concrete type

## Constraints

Constrain to exactly what the implementation uses - no more:

```typescript
// K is provably a key of T - the return type is exact
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Constrain by capability, not by class
function longest<T extends { length: number }>(a: T, b: T): T {
  return a.length >= b.length ? a : b;
}
```

## Defaults

```typescript
// Callers with the common case pass nothing
type ApiResponse<TData, TError = Error> =
  | { ok: true; data: TData }
  | { ok: false; error: TError };
```

## Inference Helpers

```typescript
// infer extracts a type the compiler already knows
type Unwrap<T> = T extends Promise<infer U> ? U : T;

// const type parameter preserves literal types
function tuple<const T extends readonly unknown[]>(...items: T): T {
  return items;
}
```

## Naming

| ❌ Soup        | ✅ Intent            |
|----------------|----------------------|
| `T, U, V`      | `TEntity, TKey`      |
| `<T = any>`    | `<T = never>` or a real default |

Single-letter `T` is fine when there is exactly one parameter and its meaning is obvious.

## Anti-Patterns

- **Generic laundering**: `<T>(x: T): T => x as T` hides an unsafe cast - fix the types
- **`any` defaults**: `<T = any>` silently disables checking at every lazy call site
- **Over-genericising**: a generic used with one concrete type is indirection, not reuse
