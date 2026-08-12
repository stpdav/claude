---
name: tailwind
description: >
  Tailwind CSS (latest stable) best practices. Use when setting up, migrating,
  or debugging Tailwind in Next.js or Node.js projects.
  Auto-activates when tailwind.config.ts is present or className="..." patterns suggest Tailwind.
allowed-tools: Read, Bash, Grep, Glob
---

# tailwind - Tailwind CSS Best Practices

## Setup Detection

```bash
# Check which version is installed
cat package.json | grep tailwind

# v4 - config via CSS, no tailwind.config.ts needed
# v3 - requires tailwind.config.ts
```

## Tailwind v4 (CSS-first config)

```css
/* app.css */
@import "tailwindcss";

@theme {
  --color-primary: #2563eb;
  --font-sans: "Inter", sans-serif;
  --radius-card: 0.75rem;
}
```

## Tailwind v3 (Config file)

```typescript
// tailwind.config.ts
import type { Config } from "tailwindcss";

export default {
  content: ["./app/**/*.{ts,tsx}", "./components/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: { primary: "#2563eb" },
    },
  },
} satisfies Config;
```

## Component Extraction

Extract repeated class patterns to components or `@apply` - never repeat long chains:

```css
/* Use @apply for repeated patterns only */
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-primary text-white rounded-md hover:bg-primary/90;
  }
}
```

## Rules

- Detect version before suggesting any config pattern - v3 and v4 differ significantly
- Never mix utility classes with CSS Modules for the same element
- Use `clsx` or `cn()` helper for conditional class logic in TSX
- Run `pnpm exec tailwindcss --input app.css --output out.css` to verify output

## Debugging

```bash
# Check generated output
pnpm exec tailwindcss -i ./src/app.css -o ./dist/out.css --watch

# Verify content paths are correct (v3)
cat tailwind.config.ts | grep content
```
