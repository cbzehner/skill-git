---
name: git
description: Advanced git workflows -- stacking, absorb, bisect, worktrees, conflict resolution, commit hygiene. Trigger-driven, layered on defaults.
allowed-tools: Bash, Read, Glob, Grep
---

# Git

Advanced git workflows. Activates for stacking, absorb, bisect, worktrees, conflict resolution, and commit hygiene. Does NOT activate for basic operations (commit, push, pull, branch, checkout, simple merge) -- those stay with Claude Code defaults.

## Trigger Table

| Trigger | Observable Signals | Excluded Signals | Risk | Escalation | Recipe |
|---------|-------------------|-----------------|------|------------|--------|
| **Post-review fixup** | Staged/unstaged changes exist AND current branch has an open PR with review comments | User is in initial implementation (no PR yet) | Low | Suggest & confirm | `fixup-history.md` |
| **Messy history before PR** | About to create PR AND commits match cleanup signals: "wip", "fix", "tmp", "squash me", "address review", duplicated prefixes, or 5+ commits that could be logically grouped | PR already exists (this is a push, not creation) | Medium | Suggest & confirm | `clean-commits.md` + `fixup-history.md` |
| **Regression debugging** | User says "when did this break", "this used to work", test was passing before, or is debugging behavior that changed | Bug is in new code the user just wrote (no regression, just a bug) | Low | Suggest & confirm | `bisect-regression.md` |
| **Large PR / split request** | User asks to split a PR, stack changes, OR PR touches 5+ files across multiple concerns | Changes are cohesive (many files in one concern is fine) | Medium | Suggest & confirm | `stack-prs.md` |
| **Parallel work needed** | User needs to switch context (hotfix, urgent bug, different feature) without losing current work | Simple branch switch with clean working tree | Low | Suggest & confirm | `parallel-worktrees.md` |
| **Merge conflict** | `git status` shows unmerged paths, or rebase/merge aborted with conflict markers | -- | High | Auto-activate (inform) | `resolve-conflicts.md` |
| **Bad rewrite recovery** | User says "I messed up", "lost my changes", history looks wrong, OR unexpected diff in range-diff output | -- | High | Auto-activate (inform) | `recover-from-mistake.md` |
| **First activation in repo** | Any trigger fires for the first time in a repo | -- | None | Inform | Config check (below) |

### Escalation Levels

- **Auto-activate (inform):** Load the recipe and begin guiding. For reactive situations (conflicts, lost work).
- **Suggest & confirm:** Propose the action and wait for user approval. Never act unilaterally on history rewrites or structural changes.
- **Inform:** Provide information (config recommendations) without suggesting action.

## Tool Detection

On first activation, detect available tools. Cache results for the session.

```bash
command -v git-absorb 2>/dev/null           # absorb
command -v gt 2>/dev/null                   # Graphite
command -v spr 2>/dev/null                  # spr
command -v wt 2>/dev/null                   # worktrunk
git --version                               # need 2.38+ for --update-refs
```

### Routing

Load the recipe for the triggered workflow. If an enhanced tool is detected, also load its tool override file. The override replaces specific steps in the recipe with enhanced equivalents.

| Domain | Recipe | Tool override (if detected) |
|--------|--------|-----------------------------|
| Fix distribution | `recipes/fixup-history.md` | `tools/absorb.md` |
| Stacking | `recipes/stack-prs.md` | `tools/graphite.md` |
| Worktrees | `recipes/parallel-worktrees.md` | `tools/worktrunk.md` |
| Bisect | `recipes/bisect-regression.md` | -- |
| Conflicts | `recipes/resolve-conflicts.md` | -- |
| Commit cleanup | `recipes/clean-commits.md` | -- |
| Recovery | `recipes/recover-from-mistake.md` | -- |

**No tools found:** All workflows work with native git. Recommend `git-absorb` as the biggest QoL win (`cargo install git-absorb`). Mention Graphite and worktrunk as optional. Never block on a missing tool.

## One-Time Repo Config Check

On first activation in a repo, check these settings:

```bash
git config --get merge.conflictStyle    # want: zdiff3
git config --get rerere.enabled         # want: true
git config --get rerere.autoUpdate      # want: true
git config --get rebase.autosquash      # want: true
git config --get rebase.autoStash       # want: true
git config --get pull.rebase            # want: true
```

If any are missing, suggest a single command block to fix all at once. Offer once, respect the answer.

### Signing Validation

If `commit.gpgsign=true` is set, verify the signing key is configured and working (`git config --get user.signingkey`). If signing is enabled but broken (key expired, agent not running), warn and offer to troubleshoot or skip. Never suggest enabling signing if it is not already configured.

### Security Audit

- `core.hooksPath` -- note if a custom hooks path is set (hook behavior is outside the skill's control)
- `credential.helper` -- verify a credential helper is configured if pushing to remotes
- Warn if the hooks directory contains scripts the user may not be aware of

## Core Policies

Always active. Non-negotiable safety rules.

### 1. Force-Push Safety

NEVER use `git push --force`. Always use `git push --force-with-lease --force-if-includes`. This prevents overwriting others' work even on "your" branch.

### 2. Force-Push Authorization

Before any force-push, check:

- Does the branch exist on the remote? (`git ls-remote --heads origin <branch>`)
- Does the branch have an open PR with reviews or comments? (`gh pr view <branch> --json reviews,comments`)
- Are there commits from other authors? (`git log --format='%ae' origin/main..<branch> | sort -u`)

If any of these indicate shared work, warn the user and require explicit confirmation. If none apply (local-only or solo unreviewed branch), proceed with `--force-with-lease`.

### 3. Pre-Destructive Backup

Before any rebase, reset, or history rewrite, create a backup:

```bash
BACKUP_BRANCH="ai-backup/$(git branch --show-current)-$(date +%Y%m%d-%H%M%S)"
git branch "$BACKUP_BRANCH"
```

Mention it to the user: "Created backup at `ai-backup/<branch>-<timestamp>`. Run `git reset --hard ai-backup/<branch>-<timestamp>` if anything goes wrong."

### 4. Verify After Rewrite

After every rebase or history surgery, run `git range-diff` against the pre-rewrite state. If something unexpected changed, flag it before proceeding.

### 5. Independent Commits Commute

When reordering commits in interactive rebase, identify which are independent (touch different files/lines) and which are dependent (touch overlapping lines). Reorder independents freely; maintain order for dependents.

### 6. Commit Messages Are Review Artifacts

The agent knows *why* the change was made. That context belongs in the commit body. Include: what prompted the change, what approach was taken and why.

### 7. Backup Cleanup

Backup branches prefixed with `ai-backup/` are recovery points. Old backups can be pruned:

```bash
git branch --list 'ai-backup/*' | xargs git branch -D
```

## References

- Recipes: `recipes/*.md` -- full native git workflows, loaded on demand per trigger
- Tool overrides: `tools/*.md` -- enhanced commands loaded in addition to recipes when tools are detected

Read recipes and tool overrides on demand when a trigger fires. Do not load them all upfront.
