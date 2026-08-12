# frameworks

Complete Claude Code setups. A framework is everything a project repo needs to make Claude
work the way I want it to: a root `CLAUDE.md`, plus a `.claude/` directory of skills,
agents, rules, and hooks.

The distinction against [`../skills/`](../skills/README.md): a skill is one capability that
follows *me* across projects and is installed at user level. A framework is a house style
that follows *a project* and is copied into its repo.

## Frameworks

| Framework | What it is |
| --- | --- |
| [`uncle-bob`](uncle-bob/README.md) | Clean code, strict typing, mandatory TDD, and governance. Architecture-neutral `core/` plus exactly one architecture overlay (currently `slices-cqs`). |

## Shape of a framework

```
<framework>/
  README.md              what it is and how to deploy it
  core/                  the parts every repo gets
    CLAUDE.md            tooling policy, skill/agent routing, rule imports
    .claude/
      skills/            per-technology standards, each a SKILL.md (+ references/)
      agents/            focused subagents (reviewers, explorer, TDD guide, ...)
      rules/             always-on rules imported by CLAUDE.md
      hooks/hooks.json   mechanical enforcement at tool-call time
  <variants>/            optional selectable profiles layered on top of core
```

Not every framework needs variants. `uncle-bob` has them because *how code is written*
(quality, typing, TDD, git) is separable from *how code is organized* (layers, boundaries,
data access), so the second is an overlay you choose per repo.

## Deploying one

Frameworks are copied, not symlinked. A project repo owns its `CLAUDE.md` and `.claude/`
so the whole team gets them from a `git clone`, with no dependency on this repo existing
on anyone's machine. That also means updates are a deliberate re-copy, not something that
silently changes under a running project.

Each framework's README has the exact steps.

## Adding a framework

Start from the shape above and keep two things true:

1. **`CLAUDE.md` routes, it does not teach.** Detail lives in skills and rules; `CLAUDE.md`
   says which of them apply and when.
2. **Rules and hooks agree.** If a rule is worth stating, prefer having a hook, a linter, or
   a CI gate enforce it, and have the rule point at that enforcement.
