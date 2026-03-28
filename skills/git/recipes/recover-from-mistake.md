# Recipe: Recover From Mistake

## When to use
Rebase went wrong, commits appear lost, history looks unexpected, range-diff showed unexpected changes.

## Escalation
Auto-activate (inform) -- this is an existing problem state.

## Recovery via backup branches
1. List backup branches:
   ```bash
   git branch --list 'ai-backup/*'
   ```
2. Find the right recovery point:
   ```bash
   git log --oneline ai-backup/<branch>-<timestamp>
   ```
3. Reset to it:
   ```bash
   git reset --hard ai-backup/<branch>-<timestamp>
   ```

## Recovery via reflog (no backup branch)
1. See recent HEAD history:
   ```bash
   git reflog --date=relative
   ```
2. Find the state before the bad operation.
3. Reset to it:
   ```bash
   git reset --hard HEAD@{N}
   ```

## Pruning old backups
- List all backup branches with their dates:
  ```bash
  git branch --list 'ai-backup/*' --format='%(refname:short) %(creatordate:relative)'
  ```
- Delete old backups when no longer needed:
  ```bash
  git branch -D ai-backup/<branch>-<timestamp>
  ```

## Failure modes

| Failure | Cause | Fix |
|---------|-------|-----|
| Reflog entry expired | Default expiry is 90 days | Recover from backup branches instead; keep backups until confident |
| Backup branch already deleted | Pruned too aggressively | Fall back to reflog recovery |
| Reset to wrong point | Picked incorrect reflog entry or backup | Re-check with `git log --oneline` before resetting; reflog still has the bad reset |

## Prevention
Link back to SKILL.md policies: always create backup before destructive ops, always verify with range-diff.
