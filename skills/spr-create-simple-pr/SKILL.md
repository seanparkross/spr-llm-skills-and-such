---
name: spr-create-simple-pr
description: Create a bare-bones GitHub PR for the current branch and open it in Chrome. No description review, no approval gate — just push, PR, done. Use when the user wants a quick PR to get changes into main without ceremony, or says things like "just open a basic PR", "quick PR", "simple PR", "no-fuss PR".
user-invocable: true
---

Push the current branch and open a minimal PR. No drafting ceremony, no approval gate — the goal is to get from "branch is ready" to "PR exists" with the fewest steps.

## Process

### Step 1 — Pre-flight checks

Run in parallel:

1. `git status` — check for uncommitted changes (never use `-uall`).
2. `git branch --show-current` — get the current branch name.
3. `git log main..HEAD --oneline` — confirm there are commits ahead of main.

**Stop if:**
- The branch is `main` — tell the user to create a feature branch first.
- There are no commits ahead of main — nothing to PR.

### Step 2 — Commit outstanding changes

If `git status` shows staged, unstaged, or untracked changes:

1. Stage relevant files (skip `.env`, credentials, large binaries).
2. Write a short commit message based on the diff.
3. Commit immediately — do not ask.

Skip if the working tree is clean.

### Step 3 — Push

1. Check upstream: `git rev-parse --abbrev-ref @{u} 2>/dev/null`.
2. No upstream → `git push -u origin $(git branch --show-current)`.
3. Upstream exists and local is ahead → `git push`.
4. Already up to date → skip.

### Step 4 — Create the PR

Use `gh pr create` directly. Do **not** invoke `spr-pr-description`.

- **Title**: the most recent commit subject if there's only one commit, otherwise a short summary of the branch's intent inferred from commit subjects. Under 72 chars, present tense imperative.
- **Body**: the bulleted list of commit subjects on this branch. That's it. No summary section, no test plan, no review notes.

```bash
gh pr create --base main --title "<title>" --body "$(cat <<'EOF'
- <commit subject 1>
- <commit subject 2>
EOF
)"
```

If there's only one commit, the body can just be that commit's subject (or omitted with `--body ""`).

Do not show the draft to the user for approval. Just create it.

### Step 5 — Open in Chrome

```bash
open -a "Google Chrome" "<pr-url>"
```

Tell the user the PR is open and include the URL.

## Rules

- No description analysis, no diff reading beyond what's needed for a commit message in Step 2.
- No "Summary / Changes / Areas affected / Test plan" template — that belongs to `spr-create-pr`.
- If the user wants a polished description later, they can run `spr-pr-description` against the PR.
