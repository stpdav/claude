---
name: nextjs
description: >
  Next.js (latest stable) best practices. Use when working on Next.js projects,
  including App Router, Server and Client Components, Route Handlers, data fetching,
  metadata, and environment configuration.
  Auto-activates when next.config.ts is present or files are in app/, server/, pages/.
allowed-tools: Read, Bash, Grep, Glob
---

# nextjs - Next.js Best Practices

## App Directory Structure

```
app/
  layout.tsx             # Root layout (always a Server Component)
  page.tsx               # Root page
  (auth)/                # Route group - no URL segment
    login/
      page.tsx
  dashboard/
    layout.tsx           # Nested layout
    page.tsx
    loading.tsx          # Suspense boundary
    error.tsx            # Error boundary (must be Client Component)

components/
  ui/                    # Shared presentational components
  features/              # Feature-specific components

lib/
  actions/               # Server Actions
  api/                   # Shared API utilities
  db/                    # Database access layer

app/api/                 # Route Handlers (REST endpoints)
  users/
    route.ts
    [id]/
      route.ts
```

## Server vs Client Components

Server Components are the default. Only add `"use client"` when the component needs:
- Browser APIs (`window`, `document`, `localStorage`)
- Event listeners (`onClick`, `onChange`)
- React hooks (`useState`, `useEffect`, `useContext`)

```tsx
// ✅ Server Component - data fetching, no interactivity (default)
// app/dashboard/page.tsx
export default async function DashboardPage() {
  const user = await db.user.findFirst();  // runs on the server
  return <UserProfile user={user} />;
}

// ✅ Client Component - only where interactivity is needed
// components/ui/Counter.tsx
"use client";

import { useState } from "react";

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

Push `"use client"` as far down the tree as possible - keep parents as Server Components.

## Data Fetching

```tsx
// ✅ Fetch in Server Components - no useEffect, no client-side waterfalls
async function UserPage({ params }: { params: { id: string } }) {
  const user = await getUser(params.id);   // data source per the architecture skill
  if (!user) notFound();
  return <UserProfile user={user} />;
}

// ✅ Parallel fetching with Promise.all to avoid waterfalls
const [user, posts] = await Promise.all([getUser(id), getPostsByAuthor(id)]);
```

Never fetch data in Client Components when the parent can be a Server Component.

Where `getUser` comes from - direct ORM access, an application-layer query handler,
or a typed API client - is the architecture overlay's decision. Follow the
`architecture` skill.

## Server Actions

Use Server Actions for mutations - no need for a separate API route for form submissions:

```tsx
// lib/actions/user.ts
"use server";

import { revalidatePath } from "next/cache";

export async function updateUser(formData: FormData) {
  const name = formData.get("name") as string;

  // Shape-validate at the boundary
  if (!name || name.length < 2) {
    return { error: "Name must be at least 2 characters" };
  }

  // The action is a thin adapter - what it dispatches to (ORM, application
  // layer, API client) is the architecture overlay's decision
  await handleUpdateUser({ name });
  revalidatePath("/dashboard");
}

// Usage in a Server Component form
// app/dashboard/settings/page.tsx
import { updateUser } from "@/lib/actions/user";

export default function SettingsPage() {
  return (
    <form action={updateUser}>
      <input name="name" />
      <button type="submit">Save</button>
    </form>
  );
}
```

## Route Handlers

Use Route Handlers for external-facing REST APIs (webhooks, mobile clients, third parties).
Not for internal data fetching - use Server Components for that.

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(
  _req: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await getUser(params.id);   // data source per the architecture skill
  if (!user) return NextResponse.json({ error: "Not found" }, { status: 404 });
  return NextResponse.json(user);
}
```

Always return typed `NextResponse.json()`. Never return raw `Response` objects.

## Metadata

```tsx
// Static metadata
export const metadata: Metadata = {
  title: "Dashboard",
  description: "User dashboard",
};

// Dynamic metadata
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const user = await db.user.findUnique({ where: { id: params.id } });
  return { title: user?.name ?? "Not found" };
}
```

Every page must export `metadata` or `generateMetadata`. Never leave pages without titles.

## Environment Variables

```typescript
// next.config.ts
const nextConfig = {
  env: {
    // Never expose server secrets to the client
  },
};

// ✅ Server-only (default - no prefix)
process.env.DATABASE_URL
process.env.SECRET_KEY

// ✅ Client-safe (must be prefixed NEXT_PUBLIC_)
process.env.NEXT_PUBLIC_API_URL
```

Validate environment variables at startup with a typed schema:

```typescript
// lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  SECRET_KEY: z.string().min(32),
  NEXT_PUBLIC_API_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

## Rules

- App Router only - do not use the Pages Router for new projects
- Default to Server Components - add `"use client"` only when required
- No data fetching in Client Components when a Server Component parent can do it
- Use Server Actions for mutations - not custom API routes for internal forms
- Data access placement (ORM vs application layer vs API client) follows the project's `architecture` skill
- Validate all environment variables at startup
- Every page exports `metadata` or `generateMetadata`
- Run `tsc --noEmit` before finishing any task

## References

- See [references/caching.md](references/caching.md) for Next.js caching strategies
- See [references/auth.md](references/auth.md) for authentication patterns (NextAuth / Auth.js)
