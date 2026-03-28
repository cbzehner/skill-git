# Recipe: Fixup History

## When to use
After addressing PR review feedback. Changes in working tree should be distributed into the prior commits they logically belong to.

## Required capabilities
Staged/unstaged changes that belong to specific prior commits.

## Escalation
Suggest & confirm — never auto-rewrite history.

## Steps (native git)
1. Stage relevant changes: `git add -p`
2. Identify target commit: `git log --oneline` to find SHA
3. Create fixup: `git commit --fixup=<target-SHA>`
4. Repeat for each group targeting different commits
5. Create backup branch (per SKILL.md policy 3)
6. Squash fixups: `git rebase -i --autosquash HEAD~N`
7. Verify: `git range-diff` (per SKILL.md policy 4)

## With git-absorb (see tools/absorb.md)
Replace steps 1-4 with: `git absorb --and-rebase`

Critical: absorb fails silently on ambiguous hunks. Check output. Handle unabsorbed hunks manually.

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Rebase conflict | Fixup touches lines modified by later commits | Resolve conflict, continue rebase |
| Absorb skips hunks | Ambiguous target commit | Use manual --fixup for those hunks |
| Wrong commit targeted | SHA identified incorrectly | Reset to backup, retry with correct SHA |

## Cleanup
Backup branch persists for recovery. No processes to kill.
