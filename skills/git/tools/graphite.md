# Tool Override: Graphite

## Detection
```bash
command -v gt 2>/dev/null
```

## Install
```bash
npm i -g @withgraphite/graphite-cli && gt auth
```

## Overrides for stack-prs.md

Replace manual branch creation and PR management:

| Native step | Graphite equivalent |
|-------------|-------------------|
| Create branch based on previous | `gt stack create` |
| Push all branches + create PRs | `gt stack submit` |
| Rebase stack after changes | `gt stack restack` |
| Visualize the stack | `gt log` |
| Land PRs in order | `gt stack land` |

### Creating a stack
```bash
gt stack create  # creates new branch in current stack
# make changes, commit
gt stack create  # another branch
# make changes, commit
gt stack submit  # pushes all, creates/updates PRs
```

### After review feedback

**CHECKPOINT: Pre-restack audit (mandatory)**

Before running `gt stack restack`, display the full stack state:

```bash
# 1. Show stack visualization with PR and CI status
gt log

# 2. List branches that will be affected
gt log --json 2>/dev/null || gt log
```

- Show every branch in the stack and its current PR/CI status
- List all branches that will be force-pushed after restacking
- If any branch has reviews, comments, or commits from other authors (per SKILL.md policy 2), warn explicitly
- **Require explicit user confirmation before proceeding**

Only after confirmation:
1. Create backup branches for the stack (per SKILL.md policy 3)
2. Run the restack:
   ```bash
   gt stack restack
   ```
3. Verify with `git range-diff` (per SKILL.md policy 4)

**CHECKPOINT: Pre-submit divergence check (mandatory)**

Before running `gt stack submit` (especially with `--force`):

```bash
# Check which branches have diverged from remote
for branch in $(gt log --json 2>/dev/null | jq -r '.[].branch' 2>/dev/null || gt log); do
  if git ls-remote --heads origin "$branch" 2>/dev/null | grep -q .; then
    local_sha=$(git rev-parse "$branch" 2>/dev/null)
    remote_sha=$(git rev-parse "origin/$branch" 2>/dev/null)
    if [ "$local_sha" != "$remote_sha" ]; then
      echo "  DIVERGED: $branch"
    fi
  fi
done
```

- Display which branches have diverged from remote
- For `--force` submits, **confirm each diverged branch before proceeding**
- Always verify the PR diff after submit: `gh pr diff <number>`

Then submit:
```bash
gt stack submit   # updates all PRs
```

### Landing

**CHECKPOINT: Pre-land verification (mandatory)**

Before running `gt stack land`:

```bash
# 1. Show stack with CI status
gt log

# 2. Verify CI on all PRs in the stack
gh pr list --head "$(git branch --show-current)" --json number,statusCheckRollup,mergeable \
  --jq '.[] | "PR #\(.number): mergeable=\(.mergeable) checks=\([ .statusCheckRollup[] | .conclusion // .status ] | join(", "))"'
```

- If any PR has failing CI checks, **warn and list failures** — do not proceed without explicit acknowledgment
- Confirm merge order is bottom-up (Graphite handles this, but verify): display the planned order
- **Require explicit user confirmation before proceeding**

Only after confirmation:
```bash
gt stack land
```

### Visualize
```bash
gt log  # shows stack with branches, PRs, status
```

## Failure Recovery

### Mid-restack conflict

When `gt stack restack` hits a conflict:

1. Graphite pauses and shows the conflicted files
2. Resolve the conflict in the affected files
3. Stage resolved files: `git add <files>`
4. Continue: `git rebase --continue` (Graphite's restack uses rebase internally)
5. After completion, verify with `gt log` and `git range-diff`

If the conflict is too complex:
```bash
git rebase --abort
# Restore from backup branches (per SKILL.md policy 3)
```

### Accidentally force-pushed wrong branch

If `gt stack submit --force` pushed stale content to a branch:

1. **Check reflog for the pre-push state**:
   ```bash
   git reflog show origin/<branch>
   ```
2. **Reset remote to the correct state**:
   ```bash
   git push --force-with-lease origin <correct-SHA>:<branch>
   ```
3. Verify the PR diff: `gh pr diff <number>`
4. Re-run `gt stack submit` to ensure Graphite's metadata is consistent

### Stack out of order after partial merge

If a PR was merged out of order (e.g., via GitHub UI instead of `gt stack land`):

1. Sync Graphite's view of the world:
   ```bash
   gt repo sync
   ```
2. Check which branches are orphaned: `gt log`
3. Restack the remaining branches:
   ```bash
   gt stack restack
   ```
4. Resolve any conflicts, then re-submit:
   ```bash
   gt stack submit
   ```
5. Verify each PR diff is clean: `gh pr diff <number>`

## When NOT to use
- Not on GitHub (Graphite is GitHub-only)
- User prefers native git workflow
- Single PR (no stack needed)
