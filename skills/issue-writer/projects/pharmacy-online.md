# pharmacy-online - PO2

Agent-side configuration for the `issue-writer` skill when working in the `pharmacy-online` repo.
This file holds what is needed to drive the API: the endpoint, the failure modes, and the rules that
are instructions to an agent rather than team conventions.

**The team conventions live in the repo**, at `docs/youtrack-project-rules.md`: workflow states,
status transitions, branch naming, PR-title format, allowed field values, and what a ticket body
should say. Read that file for anything not covered here - it is the authority. This file
deliberately does not restate it, with one exception, below, for a rule that is cheap to repeat and
expensive to get wrong.

## PR titles - where the ticket code goes

The repo's rules file remains the authority on PR titles. This one rule is repeated here because a
title is public the moment the PR opens, and fixing it afterwards means editing something colleagues
have already seen.

- **The ticket code goes at the very start of the title**, in brackets:
  `[PO2-395] Short description`.
- **The only thing that may ever precede it is a `[db-…]` tag** - `db-safe`, `db-coupled` or
  `db-breaking` - present when the change carries a runnable migration. The ticket code then goes
  immediately after it: `[db-coupled] [PO2-395] Short description`.
- **Nothing else precedes the ticket code.** Not a type prefix, not a scope, not `WIP`.
- Single space between brackets, and a single space between the final `]` and the description - no
  `-`, `:` or dash after it.

Multiple ticket codes, `WIP-N` partial deliveries and the no-dashes rule are in the repo's rules
file.

## Connection

- **Instance:** `https://track.pharmacyonline.dev`
- **MCP endpoint:** `https://track.pharmacyonline.dev/mcp` (bearer-token auth), tools exposed as
  `mcp__youtrack__*`
- **Project key:** `PO2` - pass this everywhere the skill says "the project key", and to
  `get_issue_fields_schema` before building any create or update call.

## Standing authorizations

- **`Done` on promotion.** If a ticket's delivering commit is, or at any point was, serving the
  app's live custom domain, mark it `Done` **without asking**. Verify with the live-domain check in
  the repo's `docs/vercel-deployments.md`: resolve the app's live domain via the Vercel
  `get_deployment` tool and confirm the deployed commit.
- **`Done` where no build was produced.** A merge that produces no production build has nothing to
  promote - mark it `Done` immediately.
- **Otherwise, do not mark `Done` unless the user explicitly asks.**

## Field gotchas (this instance)

The skill describes the general mechanisms. These are the specific behaviours that cost real time
here. None of them are visible to someone creating a ticket in the web UI - they exist only
because of the API.

| Symptom / trigger                                                                                             | Cause                                                                                                                                                                                                                                 | What happens                                                                                    | Do this                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `"Issue field 'Module'/'Environment' is read-only for the current user"` in `errors` / `failedToUpdateFields` | Those fields are read-only for your account **on that issue**                                                                                                                                                                         | Issue **is created / updated**; the field is silently skipped                                   | Don't block. Set it in the YouTrack UI later if needed                                                                                                                                                             |
| `MCP error -32603: Module is required`                                                                        | **`Module`'s valid values are scoped by `Subsystem`** - a value outside that subsystem's set reads back as "required", not "invalid". Also raised when `Module` is read-only on that issue but the `Subsystem` being set requires one | Atomic rejection - **the description and every other field in the same call are discarded too** | Set a `Module` from that subsystem's set (repo rules file has the pairings). If it still fails, the field is read-only on that issue: drop `Subsystem` from the call so the body still saves, and set it in the UI |
| `MCP error -32603: Environment is required`                                                                   | `Type: "Bug"` with no `Environment`, where `Environment` is read-only for you                                                                                                                                                         | Atomic rejection, nothing created                                                               | Use `Type: "Task"`, or have someone with write access file the Bug in the UI                                                                                                                                       |
| Whole `create_issue` call fails, nothing created                                                              | An enum value that is not an existing member of a fixed-enum field                                                                                                                                                                    | Atomic rejection                                                                                | Discover valid members with `search_issues` and `customFieldsToReturn: ["<Field>"]`, filtering on the field that scopes it where relevant                                                                          |
| `Assignee` rejected                                                                                           | Passed a bare string                                                                                                                                                                                                                  | Call fails                                                                                      | `Assignee` must be an **array**, even for one person: `["stephen"]`                                                                                                                                                |

**Discovery query** - the reliable way to find valid values for a scoped field:

```jsonc
{
  "query": "project: PO2 Subsystem: Dispensing",
  "customFieldsToReturn": ["Subsystem", "Module"],
}
```

## Minimal working example

Show the ticket in the conversation and get it agreed before making this call - see the skill's
"Creating an issue".

```jsonc
// create_issue - a platform/infra ticket. Subsystem "Platform" is the one value
// that does not require a Module. No Assignee: new tickets are left unassigned
// to be picked up from the backlog.
{
  "project": "PO2",
  "summary": "…",
  "description": "…", // markdown, stakeholder-framed - see "Writing ticket bodies"
  "customFields": {
    "Type": "Task",
    "State": "Backlog",
    "Priority": "Minor",
    "Subsystem": "Platform",
    "Application": ["Infrastructure"],
  },
}
```
