---
name: spr-create-pr
description: Create a GitHub PR for the current branch with a well-crafted description and open it in Chrome. Handles pre-flight checks (uncommitted changes, pushing to remote) before creating the PR. Use this skill whenever the user wants to create a PR, open a PR, submit their changes for review, or says things like "create a PR", "open a PR", "submit for review", "push and PR", or "I'm ready for review".
user-invocable: true
---

Create a GitHub PR for the current branch, handling pre-flight checks, delegating description writing to `spr-pr-description`, and opening the result in Chrome.

## Process

### Step 1 — Pre-flight checks

Run these in parallel:

1. `git status` — check for uncommitted changes (never use the `-uall` flag).
2. `git branch --show-current` — get the current branch name.
3. `git log main..HEAD --oneline` — verify there are commits ahead of main.

**Stop if:**
- The current branch is `main` — tell the user to create a feature branch first.
- There are no commits ahead of main — tell the user there is nothing to PR.

### Step 2 — Commit any outstanding changes

If `git status` shows staged, unstaged, or untracked changes:

1. Stage all relevant files (avoid secrets like `.env`).
2. Draft a concise commit message based on the diff.
3. Commit immediately — do not ask the user, just commit.

Skip this step if the working tree is clean.

### Step 3 — Push to remote

Make sure the remote branch is up to date with local:

1. Check whether an upstream tracking branch exists: `git rev-parse --abbrev-ref @{u} 2>/dev/null`
2. If no upstream: run `git push -u origin $(git branch --show-current)`.
3. If upstream exists but local is ahead: run `git push`.
4. If already up to date: skip.

### Step 4 — Create the PR description and PR

Invoke the `spr-pr-description` skill using the Skill tool. It will:

- Analyze commits and the full diff against main.
- Draft a structured PR description (summary, changes, areas affected, test plan).
- Show the draft to the user for approval.
- Create the PR via `gh pr create`.

The PR URL will be printed by `gh pr create` — capture it for the next step.

### Step 5 — Open in Chrome

Once the PR is created, open it immediately:

```bash
open -a "Google Chrome" "<pr-url>"
```

Confirm to the user that the PR is open in their browser.
