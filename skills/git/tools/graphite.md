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
```bash
# make changes, commit
gt stack restack  # rebases entire stack
gt stack submit   # updates all PRs
```

### Visualize
```bash
gt log  # shows stack with branches, PRs, status
```

## When NOT to use
- Not on GitHub (Graphite is GitHub-only)
- User prefers native git workflow
- Single PR (no stack needed)
