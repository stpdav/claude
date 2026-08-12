# skills

Standalone [Claude Code](https://claude.com/claude-code) skills, kept outside any project
repo so they apply across every project.

These are user-level: symlinked into `~/.claude/skills/` and available in every session,
whichever repo I am in. Skills that only make sense as part of a house style ship inside a
framework instead - see [`../frameworks/`](../frameworks/README.md).

## Skills

| Skill | What it covers |
| --- | --- |
| [`issue-writer`](issue-writer/SKILL.md) | Writing and maintaining issue-tracker tickets: stakeholder framing, what belongs in a body, body vs comments, open questions, voice, issue links. Tracker-independent, with the YouTrack mechanics in a final section that can be swapped. |

## Install

Claude Code loads user-level skills from `~/.claude/skills/`. Symlink each skill there:

```bash
mkdir -p ~/.claude/skills
ln -s ~/repos/claude/skills/issue-writer ~/.claude/skills/issue-writer
```

Editing the file in this repo takes effect immediately - no copy step.

## Project-specific configuration

A skill holds the portable craft only. Anything that varies by project sits in one of two places,
depending on who needs it:

- **`<skill>/projects/<project>.md`, here in this repo** - agent-side detail: endpoints, standing
  authorizations, and API failure modes. None of it is useful to someone working in a web UI, and
  some of it is personal, so it stays out of the project's own repo.
- **The project's own rules file, in its repo** - team conventions the whole team follows:
  workflow states, field values, branch and PR format, what a ticket should say. That file is the
  authority and must stand on its own, since not everyone has these skills installed.

`issue-writer/projects/pharmacy-online.md` is the worked example: it carries the MCP endpoint and the
YouTrack field gotchas, and defers everything else to that repo's `docs/youtrack-project-rules.md`.

Adding a project means adding a file under `projects/` - the skill itself does not change.
