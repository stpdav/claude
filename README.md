# claude

Everything I build for [Claude Code](https://claude.com/claude-code), in one repo: full
project frameworks at one end, single skills and commands at the other.

Nothing here is a running application. It is mostly *configuration for the agent* - skills,
agents, rules, hooks, and the standards documents that go with them - kept outside any
product repo so it can be reused, versioned, and improved in one place. Alongside that,
`mcps/` holds the setup runbooks for the MCP servers that give the agent its tools.

## Layout

```
frameworks/         complete, deployable Claude Code setups (CLAUDE.md + .claude/)
  uncle-bob/        clean code, strict typing, TDD, governance + one architecture overlay

skills/             standalone user-level skills, installed into ~/.claude/skills/
  issue-writer/     writing and maintaining issue-tracker tickets

mcps/               setup runbooks for the MCP servers I run, one per server
  google-ads-mcp.md read-only Google Ads access (GAQL), user scope
```

Two levels of granularity, and the difference matters:

- **`frameworks/`** - a whole opinionated setup for a repo. You copy it into a project and
  it brings its own `CLAUDE.md`, skills, agents, rules, and hooks. Frameworks are deployed
  *per project*.
- **`skills/`** - one self-contained capability that travels with me rather than with a
  project. Symlinked into `~/.claude/skills/`, so it is available in every session
  regardless of which repo I am in.

`mcps/` sits outside that axis: not configuration for the agent at all, but the machine
setup that gives it tools. Run once per machine, and the result lives in `~/.claude.json`.

Commands (`.claude/commands/*.md`) live wherever they belong: inside a framework when they
are part of that setup, or in a top-level `commands/` directory once there is a personal
one worth keeping. Same rule as skills - project-scoped things ship with the framework,
personal things get installed at user level.

## Frameworks

| Framework | What it is |
| --- | --- |
| [`uncle-bob`](frameworks/uncle-bob/README.md) | Clean code principles, strict typing, mandatory TDD, and governance, split into an architecture-neutral `core/` plus exactly one architecture overlay from `architectures/` (currently `slices-cqs`: vertical slices, CQS, transactional outbox). |

See [`frameworks/README.md`](frameworks/README.md) for how a framework is structured and
what deploying one involves.

## Skills

| Skill | What it covers |
| --- | --- |
| [`issue-writer`](skills/issue-writer/SKILL.md) | Writing and maintaining issue-tracker tickets: stakeholder framing, what belongs in a body, body vs comments, open questions, voice, issue links. Tracker-independent, with tracker mechanics isolated in a final section. |

See [`skills/README.md`](skills/README.md) for install instructions and the convention for
per-project configuration.

## MCP servers

| Server | What it gives Claude |
| --- | --- |
| [`google-ads-mcp`](mcps/google-ads-mcp.md) | Read-only GAQL access to Google Ads accounts (`list_accessible_customers`, `search`). Registered at user scope, so it is available in every project and never committed. |

See [`mcps/README.md`](mcps/README.md) for the shape of a guide, the registration and secret
conventions, and the servers still to be written up.

## References

Things I use alongside this repo, kept here so I stop re-finding them.

**Plugins**
- https://github.com/multica-ai/andrej-karpathy-skills/

**MCP servers** - see [`mcps/`](mcps/README.md); upstream links live there.

**Agents**
- https://github.com/multica-ai/multica
