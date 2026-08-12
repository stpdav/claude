# Next.js Caching Reference

Caching defaults changed significantly across major versions (e.g. `fetch` is uncached by
default from v15). Detect the installed version from `package.json` before applying any
pattern below - never assume.

## The Four Caches

| Cache               | Where            | What it stores                        | Invalidate with                      |
|---------------------|------------------|---------------------------------------|--------------------------------------|
| Request memoization | Server, per-request | Identical `fetch` calls in one render | Automatic - scoped to the request    |
| Data cache          | Server, persistent  | `fetch` results                       | `revalidatePath` / `revalidateTag` / time |
| Full route cache    | Server, persistent  | Rendered static routes                | Revalidating the data they use       |
| Router cache        | Client, session     | Visited route segments                | `router.refresh()`                   |

## Fetch-Level Control

```typescript
fetch(url, { cache: "force-cache" });               // opt in to the data cache
fetch(url, { cache: "no-store" });                  // always fresh
fetch(url, { next: { revalidate: 3600 } });         // ISR - refresh at most hourly
fetch(url, { next: { tags: ["users"] } });          // tag for targeted invalidation
```

## Segment-Level Control

```typescript
// app/dashboard/page.tsx - applies to the whole segment
export const dynamic = "force-dynamic";   // or "force-static"
export const revalidate = 3600;           // ISR for the segment
```

## On-Demand Invalidation

Invalidate from Server Actions after mutations - never leave stale data behind a write:

```typescript
"use server";

import { revalidatePath, revalidateTag } from "next/cache";

export async function updateUser(formData: FormData) {
  await db.user.update(/* ... */);
  revalidateTag("users");           // every fetch tagged "users"
  revalidatePath("/dashboard");     // or a specific route
}
```

## Non-Fetch Data (ORM / DB Clients)

Database calls bypass the data cache. Wrap them when caching is wanted:

```typescript
import { unstable_cache } from "next/cache";

export const getUsers = unstable_cache(
  () => db.user.findMany(),
  ["users-list"],
  { tags: ["users"], revalidate: 3600 }
);
```

## Rules

- Every mutation revalidates the paths/tags it affects - no stale reads after writes
- Prefer tags over paths - one `revalidateTag` beats hunting every affected route
- Never cache per-user data in shared caches - personalised responses are `no-store`
- If a page is unexpectedly stale or unexpectedly dynamic, check segment config before fetch options
