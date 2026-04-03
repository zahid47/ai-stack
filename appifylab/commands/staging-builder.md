# Deploy to Builder Staging

Merge current branch into `builder-test` in all 3 repos to trigger a builder staging build.

## Instructions

All 3 repos live under: `/Users/zahid47/Developer/Appifylab/EzyCourse Main/`
- `ezycourse-api`
- `courseporium-next`
- `ezy-web`

### 1. Determine the feature branch

- Get the current git branch from **each repo**.
- If **any repo is on `main`** or **on `builder-test`**: ask the user which feature branch to merge, since they're not on one.
- If the **branch names differ across repos**: show the user the branch names and ask which one(s) to merge. It's okay if not all repos have changes — ask which repos need to be included.
- If all repos are on the **same non-main, non-builder-test branch**: use that branch for all 3 repos.

### 2. For each repo that needs merging, do the following sequentially:

1. **Save the feature branch name** (e.g., `my-feature`).
2. **Stash any uncommitted changes** — run `git stash` to be safe.
3. **Delete local `builder-test`** — run `git branch -D builder-test` (ignore errors if it doesn't exist locally).
4. **Fetch latest from origin** — run `git fetch origin builder-test`.
5. **Checkout fresh `builder-test`** — run `git checkout -b builder-test origin/builder-test`.
6. **Merge the feature branch** — run `git merge <feature-branch> --no-edit`.
   - **If merge succeeds**: continue to push.
   - **If merge conflicts occur**:
     - Run `git diff --name-only --diff-filter=U` to list conflicting files.
     - Show the user the list of conflicting files.
     - **STOP and ask the user to resolve the conflicts manually.**
     - Tell them to resolve conflicts, then run `/staging-builder` again or tell you to continue.
     - Do NOT proceed with push. Do NOT abort the merge automatically.
7. **Push `builder-test`** — run `git push origin builder-test`.
8. **Checkout back to the feature branch** — run `git checkout <feature-branch>`.
9. **Pop stash if anything was stashed** — run `git stash pop` (only if step 2 stashed something).

### 3. Summary

After all repos are done, print which repos were merged and confirm the builder staging build should start automatically.

## Important

- Do NOT ask for confirmation before merging — just do it.
- Only ask the user questions when: they're on main/builder-test, branch names differ, or there are merge conflicts.
- Process repos **one at a time** so that if conflicts occur in one, the user can fix it before moving to the next.
- Always return to the feature branch after pushing, so the user's working state is preserved.
