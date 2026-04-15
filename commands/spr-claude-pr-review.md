Fetch the most recent Claude AI PR reviewer comment for the current branch's pull request, verify whether its findings are valid, fix any real issues, and post a summary comment back to the PR documenting what was fixed and what was skipped.

## Steps

### 1. Find the PR

Use `gh pr list --head <current-branch> --json number,url --jq '.[0]'` to get the PR number for the current branch. If no PR exists, tell the user and stop.

### 2. Get the last Claude bot comment

Fetch PR issue comments and filter for the Claude bot:

```
gh api repos/{owner}/{repo}/issues/{pr_number}/comments \
  --jq '[.[] | select(.user.login | test("claude|bot"; "i"))] | last'
```

If also empty, try PR review comments:

```
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --jq '[.[] | select(.user.login | test("claude|bot"; "i"))] | last'
```

If no bot comments found, tell the user and stop.

### 3. Parse the findings

The Claude PR reviewer typically structures findings with severity markers:
- Red findings (bugs, will-fail issues)
- Yellow findings (style, naming, potential issues)
- Green/informational findings

Extract each finding with its severity, file path, line number, and description.

### 4. Verify each finding against the code

For each finding, read the referenced file and line. Determine whether the finding is:
- **Valid** — the issue is real and should be fixed
- **Invalid** — the reviewer misread the code or the concern doesn't apply
- **Minor** — technically correct but not worth fixing (e.g., purely stylistic)

Present a summary to the user showing each finding, your assessment, and a brief explanation of why it's valid or not.

### 5. Plan fixes for valid issues

If there are valid findings that need fixes, enter plan mode and write a plan that addresses each one. Group related fixes together. The plan should be concise — these are typically small targeted fixes, not large refactors. Once the plan is approved, apply the fixes and commit them.

If all findings are invalid or minor, tell the user and skip the plan step. Either way, proceed to Step 6 to document the disposition on the PR.

### 6. Post summary comment to the PR

After fixes are applied (or findings skipped), post a single comment back to the PR summarising how each finding was handled. Use the PR number captured in Step 1.

Build the comment body in this format:

```
## Response to Claude PR review

**Fixed:**
- `<file>:<line>` — <one-line description of the fix>

**Skipped:**
- `<file>:<line>` — <reason: invalid / minor / out of scope>
```

Omit any section with no entries. If every finding was skipped, include only the Skipped section so the reviewer can see the feedback was considered. If there were no findings at all, do not post a comment.

Show the drafted comment body to the user in the conversation, then post it with:

```
gh pr comment <pr-number> --body "$(cat <<'EOF'
<comment body here>
EOF
)"
```

Report the comment URL returned by `gh pr comment` back to the user.
