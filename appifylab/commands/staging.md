# Deploy to Staging

Create PRs in both ezycourse-api and courseporium-next with staging labels.

## Arguments

$ARGUMENTS — optional school_id number (defaults to 273)

## Available Staging Schools

| school_id | School |
|-----------|--------|
| 273 | demo1.ezycourse.com (default) |
| 2456 | — |
| 2555 | mailer.ezycourse.com |
| 2799 | mudassir vai |
| 2903 | ezyappteam (app test only) |
| 3425 | btiger.ezycourse.com |
| 4099 | demo4.ezycourse.com |
| 4107 | basic.ezycourse.com |
| 4108 | unlimited.ezycourse.com |
| 4110 | elite1.ezycourse.com |
| 4243 | ezypeazy.ezycourse.com |
| 5189 | newdemo7.ezycourse.com |
| 5636 | Amran Bhai |
| 5671 | builder-staging.ezycourse.com |
| 5726 | Rakib Vai |
| 5737 | Sabbir Bhai |
| 5922 | Labib Vai |

## Instructions

1. Parse the school_id from "$ARGUMENTS".
   - If empty or blank, **show the user the table above** and ask them to pick a school_id (mention 273 is the default, so they can just say "y" or press enter for 273).
   - If the school_id is **not in the table above**, show the user the table and ask them to pick a valid one.
   - Do NOT proceed until you have a valid school_id.
2. Both repos live under: `/Users/zahid47/Developer/Appifylab/EzyCourse Main/`
   - Backend: `ezycourse-api`
   - Frontend: `courseporium-next`
3. Get the current git branch from **ezycourse-api** and **courseporium-next**.
4. Check for issues:
   - If **either repo is on `main`**: ask the user if they want to create a new branch. If yes, ask for a branch name, then create and checkout that branch in **both** repos.
   - If the **branch names differ**: tell the user the branch names don't match and ask if they want to create a new branch in both repos. If yes, ask for a branch name, then create and checkout that branch in **both** repos.
   - If both repos are on the **same non-main branch**: proceed.
5. For **each repo**, push the branch to origin if not already pushed (`git push -u origin <branch>`).
6. For **each repo**, check if a PR already exists for this branch → `gh pr list --head <branch> --state open`.
7. **If a PR already exists**:
   - In ezycourse-api: make sure label `STAGING-<school_id>` is on it → `gh pr edit <number> --add-label "STAGING-<school_id>"`
   - In courseporium-next: make sure label `STAGING` is on it → `gh pr edit <number> --add-label "STAGING"`
8. **If no PR exists**:
   - In ezycourse-api: `gh pr create --base main --head <branch> --title "<branch>" --body "" --label "STAGING-<school_id>"`
   - In courseporium-next: `gh pr create --base main --head <branch> --title "<branch>" --body "" --label "STAGING"`
9. Print both PR URLs at the end.

## Important

- Do NOT ask for confirmation on PR creation — just do it.
- Only ask the user questions when branches need to be created (on main or mismatched branches).
- Always confirm the push succeeded before creating the PR.
