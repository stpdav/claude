# mcps

Setup guides for the [MCP](https://modelcontextprotocol.io) servers I run in
[Claude Code](https://claude.com/claude-code), one file per server.

No server code lives here. An MCP server is somebody else's binary; what is worth keeping is
the part that took an afternoon to get right - which auth flow it actually wants, which
install route does not break on this machine, which scope to register it at, and what the
errors mean when it silently fails to connect.

The distinction against the rest of the repo: [`../skills/`](../skills/README.md) and
[`../frameworks/`](../frameworks/README.md) are configuration *for the agent*. These are
runbooks *for me*, run once per machine, and what they produce lands in `~/.claude.json`
rather than in any repo.

## Guides

| Server | What it gives Claude | Guide |
| --- | --- | --- |
| Google Ads | Read-only GAQL access to Ads accounts: `list_accessible_customers` and `search`. No writes of any kind. | [`google-ads-mcp.md`](google-ads-mcp.md) |

## Shape of a guide

Intro (what you get, and what it explicitly cannot do), prerequisites, numbered steps ending
in registration and a live test, a symptom/cause/fix troubleshooting table, and a note on
where each secret ends up.

Two things every guide states outright, because both are easy to get wrong and expensive to
debug later:

1. **The scope it registers at.** Default to `--scope user`, which stores the server in
   `~/.claude.json`: available in every project, never committed, nothing the team inherits.
   A project-scoped `.mcp.json` entry is a deliberate choice - it means the whole team gets
   the server, and secrets have to stay out of it.
2. **Where the credentials live.** Path, permissions, and expiry behaviour. Breakage a week
   after setup is usually a refresh token quietly expiring, not the server.

## Conventions

- **Install into a venv or a pinned binary, not a per-launch fetch.** Anything that clones or
  resolves a package at launch risks blowing the MCP handshake timeout, which surfaces as an
  unhelpful `CONNECTION_CLOSED`.
- **Smoke-test the command outside Claude first.** A working stdio server sits silently on
  stdin; errors print immediately. That separates "server is broken" from "registration is
  wrong".
- **Prefer read-only where the server offers it**, and say so in the intro.
- **Verify with `claude mcp list` and `/mcp`.** The first confirms what is registered and with
  which command, the second confirms it actually connected.

## Servers not yet written up

- YouTrack - https://www.jetbrains.com/help/youtrack/server/model-context-protocol-server.html
- Stape (GTM) - https://stape.io/solutions/mcp-server-for-gtm
- Supabase - https://supabase.com/docs/guides/ai-tools/mcp
- Vercel - https://vercel.com/docs/agent-resources/vercel-mcp
