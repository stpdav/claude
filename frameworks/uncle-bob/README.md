# uncle-bob

Claude Code project framework: clean code principles, strict typing, TDD, and
governance - plus a selectable architecture.

Target stack: TypeScript-only, Node/Next.js/React, pnpm, Nx monorepos, ticket-prefixed
Conventional Commits.

## Structure

```
core/                    # universal standards - copied into EVERY repo
  CLAUDE.md              # tooling policy, skills/agents routing, rules, hooks
  .claude/               # skills, agents, rules, hooks

architectures/           # implementation profiles - deploy exactly ONE
  slices-cqs/            # vertical slices + CQS + transactional outbox
    ARCHITECTURE.md      # the position: rationale, layout, responsibilities
    .claude/             # the `architecture` skill + architecture rules
```

The core is architecture-neutral by design: it enforces *how code is written*
(quality, typing, TDD, git, tickets), while an overlay decides *how code is
organized* (layers, boundaries, data access). Future overlays (e.g.
`clean-architecture/`) follow the same shape.

## What core ships

**Skills** (`core/.claude/skills/`) - the standards, loaded by task:

| Skill | Applies when |
| --- | --- |
| `quality` | Always. Design patterns, formatting, structure, naming, clean code. |
| `ts` | Any `.ts` / `.tsx` / `.js` / `.jsx` file. Strict typing, tsconfig, module resolution. |
| `react` | Any `.tsx` / `.jsx` file. Components, hooks, reactivity, performance. |
| `nextjs` | Next.js projects. App Router, Server/Client Components, route handlers, caching, auth. |
| `nx` | Nx workspaces. Project layout, module boundaries, affected tasks, generators. |
| `css` | `.css` and CSS Modules. Tokens, cascade layers, container queries. |
| `tailwind` | Tailwind projects. Setup, migration, debugging. |
| `docker` | Dockerfiles and Compose. Image hygiene, secrets, multi-stage builds. |
| `governance` | Architecture decisions, ADRs, CI/CD gates, TDD policy, shared contracts. |
| `create-pr` | `/create-pr` - drafts title and body from the branch's commits, pushes, opens the PR. |

**Agents** (`core/.claude/agents/`) - `explorer` (read-only codebase Q&A),
`code-reviewer`, `ts-reviewer`, `react-reviewer`, `tdd-guide` (RED → GREEN → REFACTOR),
`governance`, `teacher`.

**Rules** (`core/.claude/rules/`) - always-on, imported by `CLAUDE.md`:
`quality.md`, `testing.md`, `git-workflow.md`.

**Hooks** (`core/.claude/hooks/hooks.json`) - mechanical enforcement rather than
good intentions. Blocks direct pushes to `main`/`master`, blocks `git push --force`,
blocks `npm`/`yarn`/`bun` installs in favour of pnpm; warns on files over 300 lines,
plain-JS files, stray `console.log`, branches without a ticket ID, and committing or
pushing without running checks; prints a session recap on stop.

## Deploying to a repo

1. Copy `core/CLAUDE.md` and `core/.claude/` into the repo root
2. Pick ONE architecture and copy its overlay on top:
   - `architectures/<name>/.claude/` merges into the repo's `.claude/`
   - `architectures/<name>/ARCHITECTURE.md` goes to the repo root
3. Done - `CLAUDE.md`'s Architecture Overlay section picks up the overlay's
   `architecture` skill and rules automatically

Hooks are declared in `.claude/hooks/hooks.json`; wire them into the repo's
`.claude/settings.json` if the project does not pick them up automatically.

## Overlay contract

Every architecture overlay must ship four things:

1. An `architecture` skill (`.claude/skills/architecture/SKILL.md`)
2. An architecture rules file (`.claude/rules/architecture.md`)
3. Review-checklist additions (inside its skill, so reviewers pick them up)
4. An `ARCHITECTURE.md` stating the position and its assumptions

### `slices-cqs`

The one overlay so far. Its position, in short: code that changes together stays together
(the slice is the unit of organization), edges are thin and hold no database credentials,
every boundary has a typed contract, commands are atomic (aggregate plus outbox in one
transaction), and everything external leaves asynchronously through the outbox.
`architectures/slices-cqs/ARCHITECTURE.md` carries the full rationale and marks which
assumptions are still awaiting ratification.
