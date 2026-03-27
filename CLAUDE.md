# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Git hook scripts that use the `claude --print` CLI to automatically review outgoing commits and update `CLAUDE.md` in target repos when architecturally significant changes are detected.

## Structure

- `pre-push` — standalone bash script: reviews commits via Claude and auto-amends `CLAUDE.md` updates into the last commit
- `pre-commit` — standalone bash script: checks staged files for debug statements and enforces conventional commit message format
- `README.md` — installation instructions and behaviour documentation

## pre-push behaviour

1. Reads stdin from git (format: `<local ref> <local sha> <remote ref> <remote sha>`) to determine the commit range being pushed
2. For new branches, diffs against the merge base with `origin/HEAD` (defaults to `main`)
3. Passes the full `git diff` to `claude --print` with a prompt instructing it to edit `CLAUDE.md` only for significant changes (new commands, env vars, dependencies, architectural patterns, config changes)
4. If `CLAUDE.md` was modified, automatically stages and amends the last commit without prompting

## pre-commit behaviour

1. Scans staged files for debug statements (`console.log`, `debugger`) and blocks the commit if any are found
2. Validates the commit message against the conventional commits format (`type(scope): description`); supported types: `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `revert`
3. Exits non-zero (blocking) if either check fails

## Key design constraints

- Both scripts must be self-contained with no external dependencies beyond `bash` (and the `claude` CLI for `pre-push`)
- `pre-push` exits 0 (non-blocking) if `claude` is not found or if there is no `CLAUDE.md` in the repo root
