# Authentication Reference (Auth.js / NextAuth)

## Setup Shape

```
auth.ts                              # config: providers, callbacks, exported helpers
middleware.ts                        # optional: redirect unauthenticated traffic
app/api/auth/[...nextauth]/route.ts  # handlers
```

```typescript
// auth.ts
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [GitHub],
});

// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth";
export const { GET, POST } = handlers;
```

## Protecting Each Surface

Check auth at every layer that serves data - middleware alone is not protection:

```tsx
// Server Component
import { auth } from "@/auth";

export default async function DashboardPage() {
  const session = await auth();
  if (!session) redirect("/login");
  return <Dashboard user={session.user} />;
}
```

```typescript
// Server Action - these are public HTTP endpoints, always check
"use server";

export async function deletePost(id: string) {
  const session = await auth();
  if (!session) throw new AuthError("Not authenticated");
  await db.post.delete({ where: { id, authorId: session.user.id } });  // ownership in the query
}
```

```typescript
// Route Handler
export async function GET() {
  const session = await auth();
  if (!session) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  return NextResponse.json(await db.post.findMany({ where: { authorId: session.user.id } }));
}
```

## Middleware - Redirects Only

```typescript
// middleware.ts - UX layer, not a security boundary
export { auth as middleware } from "@/auth";

export const config = {
  matcher: ["/dashboard/:path*", "/settings/:path*"],
};
```

## Client Components

```tsx
"use client";
import { useSession } from "next-auth/react";  // requires <SessionProvider> in the layout

export function UserMenu() {
  const { data: session } = useSession();
  return session ? <Avatar user={session.user} /> : <LoginButton />;
}
```

## Rules

- Every Server Action checks the session - they are reachable without any UI
- Authorisation lives in the query (`where: { authorId: session.user.id }`), not only in an `if`
- Middleware is for redirect UX - the real check happens at the data layer
- `AUTH_SECRET` comes from the environment, validated at startup - never committed
- Session data sent to the client contains no secrets - tokens stay server-side
