# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single `pre-push` git hook script that uses the `claude --print` CLI to automatically review outgoing commits and update `CLAUDE.md` in target repos when architecturally significant changes are detected.

## Structure

- `pre-push` — the sole deliverable: a standalone bash script intended to be copied into `~/.githooks/` or `.git/hooks/` of other repos
- `README.md` — installation instructions and behaviour documentation

## Hook behaviour

1. Reads stdin from git (format: `<local ref> <local sha> <remote ref> <remote sha>`) to determine the commit range being pushed
2. For new branches, collects all commits reachable from the local SHA with no remote counterpart (`git rev-list --not --remotes`); for existing branches, collects commits ahead of the remote SHA
3. Skips any commit whose diff touches only `CLAUDE.md` (e.g. the hook's own follow-up commits) to avoid re-reviewing its own changes
4. Passes the diff alongside the current `CLAUDE.md` contents to `claude --print`, which assesses every change individually — checking removed lines against existing documentation for staleness, and added lines for undocumented behaviour
5. If `CLAUDE.md` is missing or empty, Claude is prompted to create it from scratch based on the diff
6. If `CLAUDE.md` was modified by Claude, a new follow-up commit (`chore: update CLAUDE.md [pre-push]`) is created and the user is asked to run `git push` again to include it

## Key design constraints

- The script must be self-contained with no external dependencies beyond `bash` and the `claude` CLI
- It exits 0 (non-blocking) if `claude` is not found
- If `CLAUDE.md` is missing, Claude will attempt to create it from scratch rather than skipping
