# Pre-Review Check

Run a comprehensive production-readiness check before submitting a PR for senior review. Supports single or multi-repo projects.

**CRITICAL SCOPE RULE:** All checks must ONLY report issues **introduced or caused by this branch**. Pre-existing issues (bugs, security vulnerabilities, performance problems, lint errors, missing tests, etc.) that already exist on `main` are **out of scope** and must NOT be reported. The purpose of this check is to evaluate whether *this branch* is safe to merge — not to audit the entire codebase. When in doubt, check if the issue exists on `main` first; if it does, skip it.

## Prerequisites

Before doing anything else, check that `gh` (GitHub CLI) is installed and authenticated:
1. Run `which gh` — if not found, tell the user to install it first: `brew install gh` (macOS) or see https://cli.github.com. Stop here.
2. Run `gh auth status` — if not authenticated, tell the user to run `gh auth login` first. Stop here.

## Instructions

### Step 1: Detect repos

- Check if the current working directory is a git repo. If yes, use it as the only repo.
- If not, find all immediate child directories that are git repos.
- For each repo, determine the current branch. Skip repos that are on `main` (nothing to review).

### Step 2: Gather context

For each repo, collect:
- The full diff against main: `git diff main...HEAD`
- The list of changed files: `git diff main...HEAD --name-only`
- The commit log since diverging from main: `git log main..HEAD --oneline`

If the diff is empty for a repo, skip it.

### Step 3: Run all checks in parallel

Launch **3 subagents**, **3 built-in skills**, and **3 automated checks** simultaneously. Pass each subagent the diff, changed file list, and repo path so they can read full files for context.

**Remind every subagent of the scope rule:** only report issues introduced by this branch/PR. If a vulnerability, performance issue, or code smell already exists on `main`, do not report it.

---

#### Built-in Skill 1: Code Review

Use Claude Code's built-in `/review` command by invoking it with the Skill tool (`skill: "review"`). It analyzes code correctness, project conventions, performance, test coverage, and security.

Capture its output and include it in the final summary.

---

#### Subagent 2: Performance & DB

**Scope:** Only analyze code that was added or modified in the diff. If a slow query or missing index already exists on `main`, do not report it. Only report performance issues that are **newly introduced** by this branch.

Check for:
- N+1 query patterns (e.g., queries inside loops, missing eager loading/preload)
- Missing database indexes for new queries or new WHERE/ORDER BY columns
- Expensive operations inside loops
- Large payload sizes or unbounded result sets (missing LIMIT/pagination)
- If migration files exist: safe rollback (`down` method), no destructive ops (`DROP TABLE`, `DROP COLUMN`) without clear intent, no locking operations on potentially large tables
- Unnecessary data fetching (selecting all columns when few are needed)

Output: list of issues found with file:line references, categorized by severity.

---

#### Subagent 3: Security Audit

**Scope:** Only report vulnerabilities **introduced by this branch**. If a security issue exists in code that was not touched by the diff, it is out of scope. To verify: check whether the vulnerable code appears in the diff as a new or modified line (lines starting with `+`). If the code is unchanged from `main`, skip it.

Use Claude Code's built-in `/security-review` command by invoking it with the Skill tool (`skill: "security-review"`). It runs a 3-phase analysis (repo context → comparative analysis → vulnerability assessment) and only reports findings with >80% confidence.

After receiving its output, **filter out** any findings that exist on `main` (i.e., the vulnerable code is not part of the diff). Only include findings where the vulnerability was introduced or worsened by this branch.

Capture the filtered output and include it in the final summary.

---

#### Built-in Skill 3: Code Quality (Simplify — report only)

Use Claude Code's built-in `/simplify` command, but run it in an **isolated worktree** (using the Agent tool with `isolation: "worktree"`). This lets `/simplify` make its fixes in a throwaway copy while the real working tree stays untouched.

After it completes, capture the list of changes it made (its diff) as the "report" — these are the code quality, reuse, and efficiency issues it found.

Include the findings in the final summary. At the end of the summary, tell the user: "Run `/simplify` to auto-fix the code quality issues above."

Additionally, in the same subagent or a separate one, check for (in **new/modified code only**):
- Proper TypeScript typing — no `any` abuse, no missing types for new code, proper return types
- Consistent patterns with the rest of the codebase

---

#### Automated Check 1: Leftovers

In **changed files only**, grep for:
- `console.log` (but not in logger utilities or test files)
- `console.warn`, `console.error` (if not intentional logging)
- `debugger`
- `TODO`, `FIXME`, `HACK`, `XXX`
- `// test`, commented-out code blocks (3+ consecutive commented lines)

**Scope:** Only report matches that appear on lines **added by this branch** (i.e., in the `+` lines of the diff). If a `console.log` or `TODO` already existed in the file on `main`, do not report it.

Output: list of matches with file:line.

---

#### Automated Check 2: Lint

Detect and run the project's linter on changed files only:
- Look for lint scripts in `package.json` (`lint`, `lint:check`, `eslint`)
- Or config files (`.eslintrc*`, `biome.json`, `.prettierrc*`)
- Run the linter on changed files

To only catch **new issues**: run the linter on the changed files. If the linter fails, check if the same errors exist on main for those files. Only report errors that are **new in this branch**.

Output: new lint errors only, or "pass" if clean.

---

#### Subagent 5: Tests

This should be a subagent because it needs AI reasoning to assess test coverage and adequacy.

Detect if the project has a test suite:
- Look for test scripts in `package.json` (`test`, `test:unit`, `test:e2e`)
- Look for test directories (`test/`, `tests/`, `__tests__/`, `spec/`)

If no tests exist, report "No test suite detected — skipped".

If tests exist:

1. **Run the test suite** — report pass/fail and any failing test names.
2. **Run coverage** if a coverage script exists (e.g., `test:coverage`, or try appending `--coverage` flag). Report coverage numbers.
3. **Analyze test adequacy for this PR** — read the changed files and the existing tests:
   - Were existing tests updated to reflect the changes? (e.g., if a function signature changed, did the test update?)
   - Were new tests written for new functionality? (new endpoints, new utils, new logic branches)
   - Are there obvious untested paths? (error cases, edge cases, boundary conditions)
   - Flag changed code that has **zero test coverage** as a blocker.
   - Flag new features/endpoints with no corresponding tests as a warning.

Output: test results, coverage summary, and a list of test gaps with file:line references categorized by severity.

---

#### Automated Check 3: Env & Config

In **changed files only**, scan for:
- New environment variable references (`process.env.`, `Env.get(`, `env(`)
- Check if these vars exist in `.env.example` or equivalent
- New config file changes
- Any `.env` files that shouldn't be committed

Output: list of new env vars that may need to be set in production.

---

#### Automated Check 4: Broken References

When routes, endpoints, functions, or exports are **removed or renamed** in the diff, verify that no other code in the repo (or sibling repos) still references the old name.

Steps:
1. From the diff, extract identifiers that were deleted or renamed (look for lines starting with `-` that define routes, exported functions, class methods, or API paths).
2. For each removed/renamed identifier, grep the **entire codebase** (all repos, not just changed files) for remaining references.
3. Exclude matches inside the diff's own deleted lines (they're already gone) and commented-out code.
4. Report any **active code** that still calls the removed identifier — these are broken references and should be flagged as **blockers**.

This check exists because removing a backend route that other frontend pages or services still call will cause runtime errors in production.

Output: list of broken references with file:line, or "pass" if none found.

---

### Step 4: Compile Summary

After all checks complete, present a single summary table per repo:

```
## [repo-name] — Review Summary

| Check              | Status | Issues |
|--------------------|--------|--------|
| Code Review        | ✅/⚠️/❌ | N blockers, N warnings (built-in /review) |
| Performance & DB   | ✅/⚠️/❌ | ... |
| Security           | ✅/⚠️/❌ | ... (built-in /security-review) |
| Code Quality       | ✅/⚠️/❌ | ... (built-in /simplify in worktree) |
| Leftovers          | ✅/⚠️/❌ | ... |
| Lint               | ✅/⚠️/❌ | ... |
| Tests              | ✅/⚠️/❌ | ... |
| Env & Config       | ✅/⚠️/❌ | ... |
| Broken References  | ✅/⚠️/❌ | ... |
```

Status meanings:
- ✅ = no issues found
- ⚠️ = warnings or suggestions only
- ❌ = blockers found — must fix before submitting

Then list all blockers first, then warnings, then suggestions — grouped by check.

End with a clear **READY** or **NOT READY** verdict.

If code quality issues were found by `/simplify`, add at the end:
> Run `/simplify` to auto-fix the code quality issues above.
