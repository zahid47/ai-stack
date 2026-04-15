---
name: gitsync
description: Clean up stale local git branches and sync remaining branches with main. Use when the user wants to prune merged/dead branches and update all feature branches with latest main. Supports single repo or multi-repo parent directories.
---

# Git Sync

Clean up stale local branches and sync remaining branches with main.

## Instructions

Detect the current working directory. If it's a git repo, run on that single repo. If it's a parent directory containing multiple git repos, find all immediate child directories that are git repos and run on each one.

Process each repo one at a time.

### For each repo:

#### Phase 1: Preparation

1. Checkout `main` and pull latest: `git checkout main && git pull origin main`.
2. Prune stale remote tracking refs: `git fetch --prune`.

#### Phase 2: Delete stale branches

Identify local branches (excluding `main` and `builder-test`) that are safe to delete:
- **Merged branches**: branches already merged into main → `git branch --merged main`
- **Dead remote branches**: branches whose upstream tracking branch is gone (shows `[gone]` in `git branch -vv`)

**Do NOT delete** branches that have no remote tracking branch at all — those are unpushed local work.

Show the user the list of branches to delete per repo and **ask for confirmation** before deleting.

#### Phase 3: Sync remaining branches with main

For each remaining local branch (excluding `main` and `builder-test`):

1. `git checkout <branch>`
2. Try `git merge main --no-edit`
   - **If no conflicts**: `git push origin <branch>` and move on.
   - **If conflicts**: `git merge --abort`, add this branch to a "skipped" list, move on.
3. `git checkout main` when done.

#### Phase 4: Summary

Print a summary per repo:
- **Deleted**: list of branches removed
- **Synced**: list of branches merged with main and pushed
- **Skipped (conflicts)**: list of branches that had conflicts — user should sync these manually
- **Untouched (unpushed)**: list of local-only branches that were left alone
