---
name: syncmain
description: Sync the current git branch with the latest main branch. Use when the user wants to pull latest main into their feature branch, rebase on main, or keep their branch up to date. Handles merge conflicts gracefully.
---

# Sync Main

Sync the current branch with main. Skip if already on main.

## Instructions

1. Run `git branch --show-current` to get the current branch.
2. If the current branch is `main`, print "Already on main, nothing to sync." and stop.
3. Checkout main and pull latest: `git checkout main && git pull origin main`.
4. Switch back to the original branch: `git checkout <branch>`.
5. Merge main into the branch: `git merge main --no-edit`.
   - **If no conflicts**: `git push origin <branch>` and report success.
   - **If conflicts**: `git merge --abort` and tell the user to resolve conflicts manually.
6. Print a one-line summary of what happened.
