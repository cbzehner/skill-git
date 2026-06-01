# Git

Handle advanced git workflows without turning history into a mess. The skill covers stacking, fixups, absorb, bisect, worktrees, conflict recovery, and PR cleanup.

## Use It For

- Splitting or stacking changes
- Cleaning up WIP commits before review
- Recovering from merge or rebase trouble

## Install

Clone the repo and run the installer:

```bash
git clone https://github.com/cbzehner/skill-git.git
cd skill-git
./install.sh all
```

Install targets:

- `./install.sh claude` installs to `~/.claude/skills/git`
- `./install.sh codex` installs to `~/.codex/skills/git`
- `./install.sh agents` installs to `~/.agents/skills/git`
- `./install.sh opencode` installs to `~/.config/opencode/skills/git`
- `./install.sh all --copy` copies files instead of symlinking

Manual install works too: symlink or copy `skills/git` into your agent's skills directory.

## Agent Support

This repo uses the plain `skills/git/SKILL.md` layout. Claude Code and Codex also get small plugin manifests at `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`.

Other agents can read the same `SKILL.md` file. If a host does not support a frontmatter field or tool name, ignore that field and follow the workflow text.

## Layout

```text
.claude-plugin/plugin.json
.codex-plugin/plugin.json
install.sh
skills/git/SKILL.md
README.md
LICENSE
```

## Public Notes

These repos are public. Keep private repo names, secrets, customer data, raw logs, cookies, and absolute filesystem paths out of examples.

## License

MIT
