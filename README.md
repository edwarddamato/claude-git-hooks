# claude-git-hooks

Git hooks that use [Claude Code](https://claude.ai/code) to keep your project context up to date automatically.

## Hooks

### `pre-push` — Auto-update CLAUDE.md

Before each push, reviews the outgoing commits and updates `CLAUDE.md` if any architecturally significant changes are detected (new commands, env vars, dependencies, patterns, config changes). Skips silently for trivial changes like bug fixes, UI tweaks, or test-only commits.

If Claude updates `CLAUDE.md`, you are prompted to amend your last commit before the push continues.

#### Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed and authenticated (`claude` command available in PATH)
- A `CLAUDE.md` file at the root of your repo (the hook skips repos without one)

#### Install (global — applies to all repos)

```bash
mkdir -p ~/.githooks
curl -o ~/.githooks/pre-push https://raw.githubusercontent.com/edwarddamato/claude-git-hooks/main/pre-push
chmod +x ~/.githooks/pre-push
git config --global core.hooksPath ~/.githooks
```

#### Install (single repo)

```bash
curl -o .git/hooks/pre-push https://raw.githubusercontent.com/edwarddamato/claude-git-hooks/main/pre-push
chmod +x .git/hooks/pre-push
```

#### How it works

1. Computes the diff of commits about to be pushed
2. Passes the diff to `claude --print` with instructions to only edit `CLAUDE.md` for significant changes
3. If `CLAUDE.md` is modified, shows the diff and prompts:
   - `y` — amends the last commit with the update and continues the push
   - `skip` — continues the push without committing the change
   - `N` (default) — aborts the push so you can review and commit manually

If the `claude` CLI is not found, the hook warns and exits without blocking the push.
