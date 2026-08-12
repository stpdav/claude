---
name: css
description: >
  Plain CSS and CSS Modules best practices. Use when writing CSS,
  working with design tokens, cascade layers, container queries, or
  CSS Modules in Next.js. Auto-activates on .css and .module.css files.
allowed-tools: Read, Bash, Grep, Glob
---

# css - CSS Best Practices

## Design Tokens First

```css
/* Always define tokens before using values */
:root {
  --color-primary: #2563eb;
  --color-surface: #ffffff;
  --color-text: #0f172a;

  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;

  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;

  --font-sans: "Inter", system-ui, sans-serif;
}
```

## Cascade Layers (Required in New Projects)

```css
/* Declare layer order at the top of your root stylesheet */
@layer reset, base, tokens, components, utilities;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; margin: 0; }
}

@layer components {
  .card { padding: var(--space-md); border-radius: var(--radius-md); }
}
```

## CSS Modules (Next.js)

```css
/* button.module.css - scoped = class names are hashed per component */
.button {
  background-color: var(--color-primary);
  padding: var(--space-sm) var(--space-md);
}
```

```tsx
// Import styles as a typed object - never hardcode class strings
import styles from "./button.module.css";

export function Button({ label }: { label: string }) {
  return <button className={styles.button}>{label}</button>;
}
```

## Progressive Enhancement

- Start with sensible defaults that work without JS
- Use `@supports` for newer features with fallbacks
- Use `clamp()` for fluid typography: `font-size: clamp(1rem, 2.5vw, 1.5rem)`
- Prefer `gap` over margin hacks for spacing in flex/grid

## Rules

- Never use magic pixel values - use tokens or relative units
- Avoid `!important` - fix specificity instead
- No inline styles in HTML/templates except for truly dynamic values
- Always test dark mode if `prefers-color-scheme` is supported
