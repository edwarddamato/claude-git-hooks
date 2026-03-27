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
2. Passes the diff (alongside current `CLAUDE.md` contents) to `claude --print` with instructions to assess every change individually and update `CLAUDE.md` if anything is stale or undocumented
3. If `CLAUDE.md` is modified, amends the last commit and exits 0 — git resolves the ref after the hook exits, so the amended commit is what gets pushed
4. If no changes are needed, the original push continues uninterrupted

If the `claude` CLI is not found, the hook warns and exits without blocking the push.

#### Flow

```mermaid
sequenceDiagram
    actor Dev as Developer
    participant Git as Git
    participant Hook as pre-push hook
    participant Claude as Claude CLI
    participant Remote as Remote (GitHub)

    Dev->>Git: git push
    Git->>Hook: invoke pre-push (stdin: ref + SHAs)
    Hook->>Hook: compute diff of outgoing commits
    Hook->>Claude: diff + current CLAUDE.md contents
    Claude->>Claude: assess each change individually

    alt CLAUDE.md needs updating
        Claude->>Hook: edits CLAUDE.md, prints OUTCOME: UPDATED
        Hook->>Git: git commit --amend --no-edit
        Hook->>Git: exit 0
        Git->>Remote: push amended commit (ref resolved after hook exits)
    else no changes needed
        Claude->>Hook: prints OUTCOME: NO_CHANGE
        Hook->>Git: exit 0
        Git->>Remote: push original commits
    end
```
