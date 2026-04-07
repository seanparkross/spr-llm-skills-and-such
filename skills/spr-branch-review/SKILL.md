---
name: spr-branch-review
description: Runs a code review on the current branch vs main, outputting findings directly in the conversation.
user-invocable: true
---

You are performing a local code review of the current branch against main. 

## Process

1. **Detect context**: Run `git branch --show-current` and `git log main..HEAD --oneline` to understand the branch name and scope of commits. Look for context for the purpose of the branch in the current conversation.

2. **Get the diff**: Run `git diff main...HEAD` to see all changes on the branch relative to main.

3. **Read project review rules**: Check for `REVIEW.md` in the project root — if present, read it and apply its rules during the review. Also check for `AGENTS.md` — if present, read it for project context (glossary, architecture, critical warnings).

4. **Read changed files in full**: For each file with meaningful changes in the diff, read the full file (not just the diff hunks) to understand the surrounding context, imports, and how the changed code fits into the broader module.

5. **Apply domain-specific checks** based on what the diff touches:
   - For changes touching picker/canvas code, check for performance anti-patterns (event handlers on shapes, Konva Groups, perfectDrawEnabled, stage.getIntersection, React state during zoom/pan)
   - For changes touching shared code (`src/components/`, `src/store/`, `src/types/`, `src/utils/`, `src/constants/`, `src/hooks/`), assess three-version impact — but only flag cross-version issues if self-hosted code actually imports the changed file
   - For changes touching API routes, verify auth checks and input validation
   - For database schema changes, check for migrations, nullable defaults, and cascade intent

6. **Output a structured review** directly in the conversation using the format below.

## Output Format

**Branch**: `{branch name}` ({N} commits ahead of main)

**Summary**: 1-2 sentence description of what the branch does.

**Findings** (only if issues found):
- 🔴 **[file:line]** Description of bug or critical issue. Suggested fix.
- 🟡 **[file:line]** Minor issue or improvement. Suggestion.
- 🟣 **[file:line]** Observation or pre-existing issue exposed by this change.

**Cross-Version Impact** (only if shared code changed):
Note which of the three deployment versions are affected and whether self-hosted actually imports the changed files.

If no issues are found, say "No issues found" and briefly note what you checked.

## Rules

- Focus on correctness: bugs that would break production, not style preferences
- Be specific: reference exact file paths and line numbers
- Be concise: one clear sentence per finding, not paragraphs
- Don't invent issues — if the code is solid, say so
- Don't comment on formatting, import ordering, or things a linter handles
- Don't repeat what AGENTS.md / CLAUDE.md says — just flag violations
- Before flagging a convention violation, verify the convention is actually followed elsewhere in the codebase — if existing code already violates it, the new code is following the de facto pattern, not breaking convention
- Don't flag trade-offs the author clearly made intentionally (e.g., moving a variable outside try/catch for scope access) — only flag if the trade-off introduces a real production risk
- 🟡 findings should be actionable — if you agree the risk is low, don't flag it
- Skip generated files (`dist/`, `release-builds/`), lock files (`pnpm-lock.yaml`), and `node_modules/`
