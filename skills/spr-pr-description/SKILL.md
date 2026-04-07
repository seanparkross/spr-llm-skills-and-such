---
name: spr-pr-description
description: Review, create, or update GitHub PR descriptions to be accurate and complete. Analyzes commits and diff on the current branch, then drafts or improves the PR description so reviewers (human and automated) have the context they need. Use this skill whenever the user mentions PR descriptions, wants to write or improve a PR description, is about to open a PR, or says things like "update the PR", "write a description", "check my PR description", or "prepare this for review".
user-invocable: true
---

You are drafting or improving a GitHub PR description for the current branch.

## Process

### Step 1 — Gather context

Run these in parallel where possible:

1. `git branch --show-current` — get the branch name.
2. `git merge-base main HEAD` — find where this branch diverged.
3. `git log main..HEAD --oneline` — list commits on this branch.
4. `git log main..HEAD --format='%s%n%b---'` — read full commit messages with bodies.
5. `git diff main...HEAD --stat` — file-level summary of what changed.
6. `git diff main...HEAD` — full diff (skip lock files and generated output).
7. `gh pr view --json number,title,body,url 2>/dev/null` — fetch the existing PR (if any).

If there is no PR yet, note that and continue — you will draft a fresh description.
If there is a PR, hold the existing description for comparison in Step 4.

### Step 2 — Understand the changes

Read any changed files that need more context (imports, surrounding logic, how the changed code fits into the broader module) using Read/Grep/Glob. Your goal is to understand three things:

- **What** changed — the concrete code modifications.
- **Why** it changed — infer from commit messages, code comments, branch name, conversation history.
- **What areas are affected** — which subsystems, layers, or deployment targets.

**Project-aware checks** (only if the relevant files/structure exist in this repo):

- Check for `REVIEW.md` in the project root. If present, read it to understand what reviewers care about — the description should preemptively address those concerns.
- Check for `AGENTS.md` or `CLAUDE.md` for project context (architecture, glossary, critical warnings).
- If the project has a multi-version or multi-target structure (e.g., shared code via symlinks), check whether modified shared files are actually imported by other targets before flagging cross-target impact.
- Note changes to areas reviewers typically scrutinize: performance-critical rendering, database schema, API routes/auth, security boundaries.

### Step 3 — Draft the description

Write the PR description using this template. Omit any section marked CONDITIONAL when it does not apply — do not leave empty sections.

```
## Summary

<1-3 sentences: what this PR does and why. Be specific — name the feature, fix,
or refactor. If there is a motivating problem, state it. A reviewer should
understand the PR from this section alone.>

## Changes

<Bulleted list of the concrete changes, grouped logically. Each bullet is one
clear statement. Use sub-bullets for non-obvious detail.

Synthesize across commits into the final state — do NOT list commits
chronologically. If commit 1 added X and commit 3 fixed a bug in X, just
describe the working version of X.>

## Areas affected

<Short list of which parts of the system are touched. Be specific enough that
a reviewer knows which domain checks apply. Examples:
- Picker (canvas rendering, hit detection)
- Designer (row properties panel)
- API routes (new endpoint /api/events/...)
- Database schema (new column on events table)
- Self-hosted standalone + iframe (shared component change)
>

## Cross-version impact                            <!-- CONDITIONAL -->

<Which deployment versions are affected and what the impact is. Only include
this section if shared code was modified AND other targets actually import it.>

## Notes for reviewers                              <!-- CONDITIONAL -->

<Intentional trade-offs, known limitations, performance context, or anything
the reviewer needs to avoid false flags. Examples:
- "Moved variable outside try/catch intentionally for scope access"
- "Tested with 2000-seat layout — no perf regression"
- "Migration is backwards-compatible: new column is nullable with default"
>

## Test plan

- [ ] Concrete verification step 1
- [ ] Concrete verification step 2
```

**Writing guidelines:**

- **Be concise.** A good PR description is scannable in 30 seconds.
- **Synthesize, don't summarize commits.** Describe the final state of the code, not the development journey.
- **Match the project voice.** Present tense imperative ("Add", "Fix", "Extract"), direct, technical. No conventional commit prefixes in the PR title.
- **PR title**: under 72 characters, present tense imperative, captures the main change.
- **Test plan**: base steps on actual changed behavior visible in the diff. Never fabricate generic test steps.
- **Cross-version / reviewer notes**: these sections exist to prevent the reviewer (human or automated) from flagging intentional decisions. Include them when they save a review round-trip; omit them when there is nothing non-obvious.

### Step 4 — Compare with existing description (if PR exists)

If the PR already has a description, evaluate it against the diff:

1. **Completeness** — Are all changes in the diff represented? Look for commits added after the description was written.
2. **Accuracy** — Does the description match what the code actually does, or does it describe intent that diverged from implementation?
3. **Missing context** — Is cross-version impact noted when applicable? Is there a test plan? Are intentional trade-offs documented?
4. **Staleness** — If commits were added, amended, or force-pushed since the description was written, it may reference outdated behavior.

Present your findings clearly:
- If the description is accurate and complete, say so — suggest only minor improvements if any.
- If it needs updates, show the improved version and explain what changed and why.

### Step 5 — Apply

**Always show the draft to the user before applying.** Never silently overwrite a PR description.

After the user approves:

- **Update existing PR**: Run `gh pr edit <number> --body "$(cat <<'EOF' ... EOF)"` using a HEREDOC to preserve formatting.
- **No PR yet**: Output the description for the user, and offer to create the PR with `gh pr create --title "..." --body "$(cat <<'EOF' ... EOF)"`.

## Rules

- Never fabricate information. If you cannot determine why a change was made, say so and ask the user.
- Omit conditional sections rather than filling them with "N/A" or "None".
- Do not pad with boilerplate. Every sentence should earn its place.
- If the branch has only documentation or config changes, keep the description minimal — a sentence and a short change list.
- Skip generated files, lock files, and build output when analyzing the diff.
