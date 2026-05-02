Rebase the current branch onto origin/main, with conflict inventory and guided resolution.

## Process

0. **Create a safety bookmark**: Run `git branch backup/pre-rebase-{original-branch-name}` so the current branch tip is preserved regardless of what happens during the rebase. Delete this branch only after you have confirmed the final state with me.

1. **Fetch and dry-run rebase**: Run `git fetch origin main`, then `git rebase origin/main`. Immediately inspect for conflicts. If the rebase halts with conflicts, run `git rebase --abort` to return to the pre-rebase state. This gives you a conflict inventory without changing anything.

2. **If no conflicts**: Proceed with the real rebase (`git rebase origin/main`), skip to step 4.

3. **If conflicts exist**: Present a conflict-resolution plan and ask me to approve each step before executing. For conflicts in the data layer (`schema.ts` or migration file ordering), prefer deleting the migrations added on this branch and regenerating them post-rebase. For all other conflicts, ensure no code introduced on this branch is silently dropped during resolution. Ask me about anything you are unsure of.

4. **Verify**: After the rebase is complete, run type-checking and the test suite. Report the results and ask me to approve the final state before pushing. The summary should be something like:
Rebase summary:
- 10 commits rebased onto origin/main
- 2 conflicts resolved:
  - {action taken}({reason})
  - {action taken}({reason})
- TypeScript: clean
- Tests: 100/100 passing
- Safety bookmark: backup/pre-rebase preserved

5. **Approval** If I approve the final state you can go ahead and git push with force-with-lease to the remote branch (`git push --force-with-lease`).
