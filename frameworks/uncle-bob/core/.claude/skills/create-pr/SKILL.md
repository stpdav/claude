---
name: create-pr
description: >
  Creates a GitHub Pull Request from the current branch.
  Reads commits since diverging from main, drafts a Conventional Commits-compliant
  title and structured body, pushes the branch, and opens the PR via `gh`.
  Invoke with: /create-pr
allowed-tools: Read, Bash, Grep, Glob
---

# /create-pr - Create a Pull Request

You are a Git and GitHub expert. When invoked, follow these steps exactly.

---

## Step 1 - Verify prerequisites

```bash
# Must be on a feature branch
git branch --show-current

# gh CLI must be authenticated
gh auth status
```

If the current branch is `main` or `master`, stop and tell the user:
> "You are on the main branch. Create a feature branch first: `git checkout -b type/description`"

If `gh` is not authenticated, stop and tell the user:
> "Run `gh auth login` before using /create-pr"

---

## Step 2 - Gather commit context

```bash
# Find the base branch (main or master)
git remote show origin | grep 'HEAD branch'

# Extract the ticket ID from the branch name (any tracker: ABC-123, PO-42, ...)
git branch --show-current | grep -oE '[A-Z][A-Z0-9]*-[0-9]+' | head -1

# List all commits since diverging from base
git log --oneline origin/main..HEAD
# or: git log --oneline origin/master..HEAD

# Show the full diff for context
git diff origin/main..HEAD --stat
```

If the branch name contains no ticket ID, check the commit headers for a `[TICKET-ID]` prefix.
Still nothing → proceed without one, but flag it in the final output.

---

## Step 3 - Check for uncommitted changes

```bash
git status --short
```

If there are uncommitted changes, warn the user:
> "You have uncommitted changes. Commit or stash them before creating a PR."
Stop and do not proceed.

---

## Step 4 - Draft the PR title

Rules:
- Follow Conventional Commits: `type(scope): description`
- Prefix with the ticket ID from Step 2 in brackets: `[TICKET-ID] type(scope): description`
- Max 70 characters including the prefix
- Use the dominant commit type from Step 2 (feat > fix > refactor > chore)
- Scope = the main module/area touched (optional but preferred)
- Description: imperative mood, lowercase, no period
- No ticket ID found → no prefix
- The prefix is safe in commit history, including squash-merges from the PR title -
  every repo ships the commitlint config from git-workflow.md that accepts it

Examples:
```
[ABC-123] feat(auth): add JWT refresh token rotation
[ABC-456] fix(cart): prevent negative quantities on checkout
refactor(user): extract profile update into service layer
```

---

## Step 5 - Draft the PR body

Use this exact structure:

```markdown
Ticket: <TICKET-ID - omit this line if there is no ticket>

## Summary
- <bullet 1 - what changed and why>
- <bullet 2>
- <bullet 3 if needed>

## Type of change
- [ ] Bug fix
- [ ] New feature
- [ ] Refactor
- [ ] Documentation
- [ ] Chore / tooling

## Test plan
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manually tested: <describe scenario>

## Checklist
- [ ] Linter passes (`biome check` / `eslint`)
- [ ] Type checker passes (`tsc --noEmit`)
- [ ] No secrets or debug statements committed
- [ ] Branch follows naming convention (`type/TICKET-ID-description`)
- [ ] Ticket ID present in branch name and PR title (if the work is ticketed)
```

---

## Step 6 - Push and create the PR

```bash
# Push branch with upstream tracking
git push -u origin HEAD

# Create the PR
gh pr create \
  --title "<drafted title>" \
  --body "<drafted body>" \
  --draft
```

Always create as `--draft` by default. Tell the user:
> "PR created as a draft. Mark it ready for review when all checklist items are complete."

---

## Step 7 - Output

Return:
- The PR URL
- The linked ticket ID - or a warning that no ticket ID was found on the branch or commits
- A one-line summary of what was included
- Any checklist items that still need attention based on the diff
