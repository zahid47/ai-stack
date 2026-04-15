---
name: staging
description: Deploy to staging by creating PRs in both ezycourse-api and courseporium-next with staging labels. Use when the user wants to deploy a feature branch to a staging environment for testing. Supports multiple staging schools.
---

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

## Prerequisites

Before doing anything else, check that `gh` (GitHub CLI) is installed and authenticated:
1. Run `which gh` — if not found, tell the user to install it first: `brew install gh` (macOS) or see https://cli.github.com. Stop here.
2. Run `gh auth status` — if not authenticated, tell the user to run `gh auth login` first. Stop here.

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
5. **Check if already in staging** — for **each repo**, check if a PR with the staging label already exists for this branch → `gh pr list --head <branch> --state open`.
6. **If already in staging** (PR exists with staging label):
   - For **each repo**, check for uncommitted changes (`git status --porcelain`).
   - If there are uncommitted changes: stage all changes (`git add -A`), create a commit with a short descriptive message summarizing the changes, and push to origin.
   - If there are no uncommitted changes but unpushed commits: just push to origin.
   - If fully up to date: report "already up to date".
   - Re-confirm the staging label is still on the PR (in case it was removed) and re-add if needed.
   - Print the existing PR URLs and done.
7. **If NOT in staging** (no PR exists):
   - For **each repo**, push the branch to origin if not already pushed (`git push -u origin <branch>`).
   - In ezycourse-api: `gh pr create --base main --head <branch> --title "<branch>" --body "" --label "STAGING-<school_id>"`
   - In courseporium-next: `gh pr create --base main --head <branch> --title "<branch>" --body "" --label "STAGING"`
   - Print both PR URLs at the end.

## Important

- Do NOT ask for confirmation on PR creation, commits, or pushes — just do it.
- Only ask the user questions when branches need to be created (on main or mismatched branches) or for school_id selection.
- Always confirm the push succeeded before creating the PR.
