# Async Patterns Reference

## Sequential vs Parallel

```typescript
// ❌ Sequential - each await blocks the next
const user = await getUser(id);
const posts = await getPosts(id);

// ✅ Parallel - independent work runs concurrently
const [user, posts] = await Promise.all([getUser(id), getPosts(id)]);

// ✅ Partial failure acceptable - inspect each outcome
const results = await Promise.allSettled([getUser(id), getPosts(id)]);
```

## Expected Failures - Result, Not Throw

```typescript
async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) return { ok: false, error: new Error(`HTTP ${res.status}`) };
    return { ok: true, value: await res.json() };
  } catch (err) {
    return { ok: false, error: err instanceof Error ? err : new Error(String(err)) };
  }
}
```

## Cancellation

```typescript
// Always pass a signal to fetches triggered by user input
const controller = new AbortController();
const res = await fetch(url, { signal: controller.signal });

// Later - e.g. on new keystroke or unmount
controller.abort();
```

## Timeouts

```typescript
const res = await fetch(url, { signal: AbortSignal.timeout(5_000) });
```

## Anti-Patterns

- **Floating promises**: every promise is awaited or explicitly `void`-ed
- **Promise constructor wrap**: never `new Promise(async (resolve) => ...)` - the function is already a promise
- **`async` in `forEach`**: `forEach` ignores promises - use `for...of` or `Promise.all(items.map(...))`
- **`return promise` inside `try`**: rejections escape the catch - use `return await promise`
- **Fire-and-forget mutations**: awaiting writes is not optional - unawaited failures vanish silently
