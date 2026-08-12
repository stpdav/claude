---
name: nx
description: >
  Nx monorepo best practices (latest stable). Use when working in an Nx workspace -
  project layout, module boundaries, affected-based tasks, caching, and generators.
  Auto-activates when nx.json is present.
allowed-tools: Read, Bash, Grep, Glob
---

# nx - Nx Monorepos

## When Nx

Use an Nx workspace when a repo holds more than one deployable app, or apps share
non-trivial code. A single Next.js app does not need Nx.

One workspace per product - never multiple products in one repo. Git has no
per-folder read access, so product isolation lives in per-repo permissions.
Cross-product code (design system, shared utils) ships as versioned private
packages, not shared folders.

## Workspace Layout

`apps/` hold deployables and stay thin - routing, config, composition only; the code
lives in `libs/`. The concrete layout and lib taxonomy are defined by the project's
architecture overlay - see the `architecture` skill.

```
apps/                    # deployables, thin
libs/                    # the code - structured per the architecture overlay
nx.json
pnpm-workspace.yaml
tsconfig.base.json       # single source of path aliases
```

## Module Boundaries

Tag every project; enforce the tags in CI with `@nx/enforce-module-boundaries`.
Boundary enforcement is eslint-only - keep eslint for this rule even in biome repos.
The tag taxonomy and dependency matrix come from the architecture overlay:

```jsonc
// project.json - tags per the architecture overlay's matrix
{ "tags": ["scope:...", "type:..."] }
```

## Tasks & Affected

```bash
# One project
pnpm exec nx test web
pnpm exec nx build api

# Only what a change touched - the default in CI
pnpm exec nx affected -t lint,typecheck,test,build

# See the dependency reality before restructuring
pnpm exec nx graph
```

## Caching

- The local computation cache is on by default - never disable it
- CI restores the cache (or uses Nx remote caching); a cold-cache pipeline wastes the monorepo
- Declare cacheable ops (`build`, `test`, `lint`, `typecheck`) in `nx.json` `targetDefaults`

## Generators

Scaffold with generators so every project starts consistent:

```bash
pnpm exec nx g @nx/next:app apps/web
pnpm exec nx g @nx/react:lib libs/shared/ui
pnpm exec nx g @nx/js:lib libs/shared/util
```

## Conventions

- Commit scope = project name: `[ABC-123] feat(web): ...`, `[ABC-123] fix(shared-ui): ...`
- One root `tsconfig.base.json` holds all path aliases; projects extend it
- Single version policy: shared dependencies versioned once at the root

## Rules

- `apps/` contain no business logic - extract to `libs/`
- Every project is tagged; module boundaries enforced in CI per the architecture overlay's matrix
- CI runs `nx affected`, never the full graph on every PR
- Use generators for new apps/libs - no hand-rolled project scaffolding
- The repo quality gates (type check, lint, tests, coverage) apply per affected project
