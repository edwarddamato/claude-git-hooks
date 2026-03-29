# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single `pre-push` git hook script that uses the `claude --print` CLI to automatically review outgoing commits and update `CLAUDE.md` in target repos when architecturally significant changes are detected.

## Structure

- `pre-push` — the sole deliverable: a standalone bash script intended to be copied into `~/.githooks/` or `.git/hooks/` of other repos
- `README.md` — installation instructions and behaviour documentation

## Hook behaviour

1. Reads stdin from git (format: `<local ref> <local sha> <remote ref> <remote sha>`) to determine the commit range being pushed
2. For new branches, diffs against the merge base with `origin/HEAD` (defaults to `main`)
3. Passes the full `git diff` to `claude --print` with a prompt instructing it to edit `CLAUDE.md` only for significant changes (new commands, env vars, dependencies, architectural patterns, config changes)
4. If `CLAUDE.md` was modified, creates a new follow-up commit (`chore: update CLAUDE.md [pre-push]`), exits 1 to cancel the original push, and instructs the user to run `git push` again to include the new commit

## Key design constraints

- The script must be self-contained with no external dependencies beyond `bash` and the `claude` CLI
- It exits 0 (non-blocking) if `claude` is not found or if there is no `CLAUDE.md` in the repo root
