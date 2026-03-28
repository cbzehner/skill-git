# Git

Advanced git workflows for Claude Code. Stacking, absorb, bisect, worktrees, conflict resolution, and commit hygiene -- trigger-driven, layered on top of defaults.

> *Native git is first-class. Enhanced tools are optional power-ups.*

## Why Use This?

- **Trigger-driven**: Activates at the right moment -- post-review fixup, messy history, regression debugging, merge conflicts
- **Detect and adapt**: Finds installed tools (git-absorb, Graphite, worktrunk) and uses them when available
- **Safety-first**: Every destructive operation gets a backup branch, force-push always uses `--force-with-lease`
- **Native git works**: Every workflow works with plain git. No extra tools required.

## Prerequisites

All optional. The skill works with native git alone.

| Tool | Install | Enhances |
|------|---------|----------|
| **git-absorb** | `cargo install git-absorb` | Distributing review fixes into prior commits |
| **Graphite** | `npm i -g @withgraphite/graphite-cli && gt auth` | Stacked PRs and stack management |
| **worktrunk** | `cargo install worktrunk` | Worktree management and parallel work |

Requires git 2.38+ for `--update-refs` support in stacking workflows.

## Installation

### From Marketplace

```bash
# Add the marketplace
/plugin marketplace add cbzehner/skill-git

# Install the skill
/plugin install git@cbzehner
```

### Manual Installation

Clone into your `.claude/skills/` directory:

```bash
cd ~/.claude/skills/
git clone https://github.com/cbzehner/skill-git.git git
```

## Usage

The skill activates automatically for advanced git tasks:

```
You: I need to distribute these review fixes into the right commits
You: Clean up this branch history before I create a PR
You: This test used to pass -- when did it break?
You: Split this PR into a stack
You: I need to work on a hotfix without losing my current work
You: I messed up this rebase, help me recover
```

Basic git operations (commit, push, pull, branch, checkout) stay with Claude Code defaults.

## Files

```
git/
├── SKILL.md                    # Main skill definition (triggers, policies, safety, config)
├── recipes/
│   ├── fixup-history.md        # Distribute review fixes into prior commits
│   ├── stack-prs.md            # Split work into stacked PRs
│   ├── bisect-regression.md    # Automated binary search for regressions
│   ├── resolve-conflicts.md    # Rebase conflicts + generated file regeneration
│   ├── clean-commits.md        # Reorder, squash, commit message hygiene
│   ├── parallel-worktrees.md   # Isolated parallel work
│   └── recover-from-mistake.md # Reflog, undo log, backup branches
├── tools/
│   ├── absorb.md               # git-absorb overrides for fixup distribution
│   ├── graphite.md             # Graphite overrides for stacking
│   └── worktrunk.md            # worktrunk overrides for worktrees
├── README.md                   # This file
└── LICENSE                     # MIT
```

## License

MIT
