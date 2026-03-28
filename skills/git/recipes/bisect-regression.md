# Recipe: Bisect Regression

## When to use
Something used to work and now doesn't. Before speculating about the cause, let bisect find the exact commit.

## Escalation
Suggest & confirm — propose bisect and wait for user approval before starting.

## Steps
1. Identify boundaries:
   - Bad: `HEAD` (current broken state)
   - Good: ask user, check CI, or `git log --oneline` for a known-good commit
2. Write a deterministic test script OUTSIDE the repo:
   ```bash
   cat > /tmp/bisect-test.sh << 'SCRIPT'
   #!/bin/bash
   # Exit 0 = good, 1-124 = bad, 125 = skip (can't test this commit)
   cd "$1"
   # Test the specific behavior, not the entire suite
   npm test -- --grep "specific test name" 2>/dev/null
   SCRIPT
   chmod +x /tmp/bisect-test.sh
   ```
3. Run automated bisect:
   ```bash
   git bisect start HEAD <good-SHA>
   git bisect run /tmp/bisect-test.sh "$(pwd)"
   ```
4. Record the culprit commit, then reset:
   ```bash
   git bisect reset
   ```
5. Report the guilty commit with its diff and message.

## Key rules for bisect scripts
- **Self-contained** — working tree changes per commit, script must not depend on it
- **Place in /tmp** — use absolute path, never inside the repo
- **Exit codes matter** — 0=good, 1-124=bad, 125=skip
- **Specific test** — test the exact broken behavior, not the full suite

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Build fails on old commits | Dependencies or tooling changed | Use `exit 125` to skip untestable commits |
| Test script not self-contained | Script references files in working tree | Move all logic into `/tmp/bisect-test.sh` with no repo-relative paths |
| Wrong good/bad boundaries | Regression older than assumed | Widen the range with an earlier known-good commit |

## Cleanup
Always run `git bisect reset` to return to original state. Remove `/tmp/bisect-test.sh`.
