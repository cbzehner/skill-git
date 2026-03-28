# Recipe: Stack PRs

## When to use
Work touches multiple concerns that should be reviewed independently, or a PR is too large.

## Escalation
Suggest & confirm — never restructure branches without approval.

## Steps (native git, requires Git 2.38+)

### Creating a stack from scratch
1. Identify logical concerns in the work
2. Create branch for first concern off main:
   ```bash
   git checkout -b feat/db-schema main
   # cherry-pick or rebase relevant commits
   ```
3. Create branch for next concern off previous branch:
   ```bash
   git checkout -b feat/api feat/db-schema
   # cherry-pick or rebase the next set
   ```
4. Repeat for each concern
5. Push each branch, create separate PRs noting dependencies in descriptions

### Splitting an existing branch
1. Create backup branch (per SKILL.md policy 3)
2. `git rebase -i main` — reorder commits by concern
3. Note SHA boundaries between concerns
4. Create branches at split points:
   ```bash
   git branch feat/first-concern <SHA-at-boundary>
   git branch feat/second-concern <SHA-at-next-boundary>
   ```
5. Push each as separate PR

### Keeping the stack in sync
```bash
git rebase --update-refs main
```
Rebases the entire stack and updates all branch pointers in one operation.

### After review changes
Rebase the stack again with `--update-refs` to propagate changes through all dependent branches.

### Landing order
Land bottom-up (base branch first). After each merge, rebase remaining branches onto the updated main.

## With Graphite (see tools/graphite.md)
Replace manual branching with `gt` commands: `gt stack create`, `gt stack submit`, `gt stack restack`, `gt stack land`.

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Conflict during restack | Overlapping changes between stacked branches | Resolve conflicts, `git rebase --continue`, then re-run `--update-refs` |
| Orphaned branches | Base branch merged but dependents not rebased | Rebase orphaned branches onto updated main |
| Wrong merge order | Merged a dependent before its base | Rebase the dependent onto main, resolve conflicts, re-push |

## Cleanup
Remove merged branches: `git branch -d feat/db-schema feat/api`. Prune stale remote references: `git fetch --prune`.
