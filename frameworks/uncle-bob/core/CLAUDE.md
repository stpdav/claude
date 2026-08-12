# uncle-bob - Claude Code Configuration

> Clean code principles, strict typing, TDD, and governance for every session.

## Version & Tooling Policy

Never hardcode version numbers. Always detect from `package.json` and lockfiles.
Use web search to verify latest stable patterns when uncertain.

| Technology  | Version Policy  | Primary Tool | Fallback    |
|-------------|-----------------|--------------|-------------|
| Node.js     | Latest LTS      | pnpm         | npm / node  |
| TypeScript  | Latest stable   | via pnpm     | -           |
| React       | Latest stable   | via pnpm     | -           |
| Next.js     | Latest stable   | via pnpm     | -           |
| Nx          | Latest stable   | via pnpm     | -           |
| Tailwind    | Latest stable   | -            | -           |
| Docker      | Latest stable   | docker       | -           |

---

## Mandatory Skills

Always apply `quality` on every task - no exceptions.

Apply `ts` on every task touching `.ts` / `.tsx` / `.js` / `.jsx` files.
Apply `react` on every task touching `.tsx` / `.jsx` files.
Apply `nextjs` on every task in a Next.js project (`next.config.ts` present, or files in `app/`).
Apply `nx` on every task in an Nx workspace (`nx.json` present).
Apply `css` on every task touching `.css` files or CSS Modules.
Apply `tailwind` on every task in a project using Tailwind (`tailwind.config.ts` present, or `className` utility patterns).
Apply `docker` on every task touching `Dockerfile`, `docker-compose*.yml`, or `.dockerignore`.

---

## Architecture Overlay

Every deployed repo carries exactly one architecture overlay from `architectures/`
(e.g. `slices-cqs`). The overlay supplies the `architecture` skill and
`.claude/rules/architecture.md` - this core is architecture-neutral by design.

Always apply the `architecture` skill on tasks touching application, domain, or
infrastructure code.

@.claude/rules/architecture.md

---

## Agents

Use these subagents for focused, delegated work:

| Agent              | When to use                                                                      |
|--------------------|----------------------------------------------------------------------------------|
| `explorer`         | Read-only codebase Q&A before making changes                                     |
| `code-reviewer`    | After any implementation - quality, security, maintainability                    |
| `tdd-guide`        | When writing new features - enforce RED→GREEN→REFACTOR                           |
| `governance`       | Architecture decisions, ADRs, repo gates, TDD policy                             |
| `teacher`          | Explain code in plain language, beginner-friendly walkthroughs                   |
| `ts-reviewer`      | After touching `.ts` / `.tsx` / `.js` files                                      |
| `react-reviewer`   | After touching `.tsx` / `.jsx` files or Next.js pages, layouts, and actions      |

---

## Commands

| Command       | What it does                                                                    |
|---------------|---------------------------------------------------------------------------------|
| `/create-pr`  | Draft a Conventional Commits title + structured body and open a draft PR via `gh` |

---

## Rules (always follow)

- Follow `quality` standards on every file touched
- TypeScript only: write `.ts` / `.tsx`, never plain `.js` / `.jsx` - JS is allowed only for configs that cannot load TS
- TDD: write failing test first, then implementation, then refactor
- Ticketed work carries its ticket ID (any tracker, `ABC-123` pattern): branch `feat/ABC-123-...`, commit headers and PR titles prefixed `[ABC-123] type(scope): ...`
- Never commit secrets, hardcoded credentials, or API keys
- Keep files under 300 lines - split if larger
- Use `pnpm` for package management and running scripts by default
- Run linter + type checker before considering a task done
- Document public APIs and exported functions

Full rule files (loaded into every session):

@.claude/rules/quality.md
@.claude/rules/testing.md
@.claude/rules/git-workflow.md

---

## Hooks

- `pre-commit`: remind to run linter + type checker + tests before committing
- `post-edit`: warn if file exceeds 300 lines; warn on leftover `console.log` in `.ts` / `.tsx` / `.js`; warn when a plain `.js` / `.jsx` file is written (TypeScript only)
- `pre-bash`: block package-manager mixing (`npm install`, `bun add`, `yarn add` - use `pnpm add`); warn when a new branch is created without a ticket ID
- `pre-push`: block direct push to `main`/`master` and force pushes; remind to run checks first
- `stop`: after every session, print a recap of changes and project state

---

## Session Recap (mandatory final step)

After every task that modifies files, Claude must output:

### What was done
- Bullet list of every file created or modified, with one line explaining the change

### Project state
- Current branch and last commit
- Any uncommitted changes remaining
- Files approaching the 300-line limit (if any)
- TODOs or follow-up actions identified during the session
