---
description: Finds all open Dependabot PRs on the current repository, combines them into a single branch, runs validation (tsc, tests, Docker build), and creates a combined PR. Use when the user says "merge dependabot", "combine dependabot PRs", "dependabot", or "update dependencies".
user-invocable: true
---

# Merge Dependabot PRs

Consolidates all open Dependabot PRs into a single tested branch and PR.

---

## Phase 0: Setup Permissions

Before doing anything else, temporarily grant auto-approval for the commands this skill needs.

1. Check if `.claude/settings.json` already exists in the project root.
   - If it does, read it and store its contents so it can be restored later.
   - If `.claude/` directory doesn't exist, create it.
2. Write `.claude/settings.json` with the following content (merging with existing settings if the file already existed — preserve all existing keys and only add/merge the `permissions.allow` entries):

```json
{
  "permissions": {
    "allow": [
      "Bash(git status *)",
      "Bash(git fetch *)",
      "Bash(git checkout -b *)",
      "Bash(git checkout *)",
      "Bash(git merge *)",
      "Bash(git merge --abort)",
      "Bash(git push -u origin *)",
      "Bash(git add *)",
      "Bash(gh pr list *)",
      "Bash(gh pr create *)",
      "Bash(gh repo view *)",
      "Bash(npm test*)",
      "Bash(npm install*)",
      "Bash(yarn install*)",
      "Bash(pnpm install*)",
      "Bash(npx tsc *)",
      "Bash(npx eslint *)",
      "Bash(docker build *)",
      "Bash(docker rmi *)"
    ]
  }
}
```

3. Add `.claude/settings.json` to `.git/info/exclude` so it won't show up in git status or get committed. Only add the line if it's not already present.

---

## Phase 1: Discovery

1. Confirm the current repo has a GitHub remote:
   ```bash
   gh repo view --json nameWithOwner -q '.nameWithOwner'
   ```
2. List all open Dependabot PRs:
   ```bash
   gh pr list --author "app/dependabot" --state open --json number,title,headRefName,baseRefName
   ```
3. If no Dependabot PRs are found, inform the user and stop.
4. Print a summary table of found PRs (number, title, branch) and confirm with the user before proceeding.

---

## Phase 2: Branch Setup

1. Ensure the working tree is clean (`git status --porcelain`). If not, ask the user to commit or stash first.
2. Fetch latest from origin:
   ```bash
   git fetch origin
   ```
3. Determine the default branch (usually `main` or `master`):
   ```bash
   gh repo view --json defaultBranchRef -q '.defaultBranchRef.name'
   ```
4. Create a new branch from the default branch:
   ```bash
   git checkout -b chore/dependabot-combined-YYYY-MM-DD origin/<default-branch>
   ```
   Use today's date in the branch name.

---

## Phase 3: Merge Dependabot Changes

For each Dependabot PR, in order:

1. Fetch the PR's branch:
   ```bash
   git fetch origin <pr-branch>
   ```
2. Merge it into the combined branch:
   ```bash
   git merge origin/<pr-branch> --no-edit
   ```
3. If a merge conflict occurs:
   - First, attempt to resolve it automatically if it's a lockfile conflict (run the appropriate package manager install/lock command and stage the result).
   - If the conflict is in source files, inform the user which PR caused the conflict, show the conflicting files, and ask how to proceed (skip this PR or resolve manually).
   - If skipping, run `git merge --abort` and continue with the next PR.
4. Track which PRs were successfully merged and which were skipped.

---

## Phase 4: Validation

Run the following checks sequentially. Stop and report on the first failure.

### 4a. TypeScript Check (if applicable)

Only run if a `tsconfig.json` exists in the project root.

```bash
npx tsc --noEmit
```

If tsc fails, analyze the errors. If they are type errors caused by the dependency updates:
- Attempt to fix them.
- Re-run tsc to confirm.
- If unable to fix, report the errors to the user and ask how to proceed.

### 4b. ESLint Check (if applicable)

Only run if an ESLint config exists in the project (`.eslintrc.*`, `eslint.config.*`, or an `eslintConfig` key in `package.json`).

```bash
npx eslint .
```

If ESLint fails, analyze the errors. If they are caused by the dependency updates (e.g. new rule defaults from an updated plugin):
- Attempt to fix automatically with `npx eslint . --fix`.
- Re-run ESLint to confirm.
- If unable to fix, report the errors to the user and ask how to proceed.

### 4c. Test Suite

Detect the test runner from `package.json` scripts and run the test suite:

```bash
npm test
```

Or the equivalent (`yarn test`, `pnpm test`, etc.) based on the lockfile present.

If tests fail:
- Analyze the failures to determine if they're related to dependency updates.
- Attempt to fix if straightforward.
- If unable to fix, report to the user and ask how to proceed.

### 4d. Docker Image Build (if applicable)

Only run if a `Dockerfile` exists in the project root.

```bash
docker build -t dependabot-validation-test .
```

If the build fails, report the error and ask the user how to proceed.

After successful validation, clean up the test image:
```bash
docker rmi dependabot-validation-test 2>/dev/null || true
```

---

## Phase 5: Create PR

1. Push the combined branch:
   ```bash
   git push -u origin chore/dependabot-combined-YYYY-MM-DD
   ```
2. Create the PR using `gh`:
   ```bash
   gh pr create \
     --title "chore(deps): combine dependabot updates YYYY-MM-DD" \
     --body "$(cat <<'EOF'
   ## Summary

   Combines the following Dependabot PRs into a single update:

   <list each merged PR as "- #NUMBER TITLE">

   <if any PRs were skipped, add:>
   ### Skipped PRs (merge conflicts)
   <list each skipped PR as "- #NUMBER TITLE — reason">

   ## Validation

   - [x] TypeScript compilation (or N/A)
   - [x] ESLint (or N/A)
   - [x] Test suite
   - [x] Docker build (or N/A)

   Closes <list each merged PR number as #NUMBER separated by commas>
   EOF
   )"
   ```
3. Print the PR URL to the user.

---

## Phase 6: Cleanup Permissions

**This phase MUST run regardless of whether the skill succeeded or failed.** Even if an error occurs in any earlier phase, always execute this cleanup.

1. If `.claude/settings.json` existed before Phase 0:
   - Restore it to its original contents (the version saved in Phase 0 step 1).
2. If `.claude/settings.json` did NOT exist before Phase 0:
   - Delete `.claude/settings.json`.
   - If `.claude/` directory is now empty, delete it too.
3. Remove the `.claude/settings.json` line from `.git/info/exclude` if it was added in Phase 0.

---

## Important Rules

- **Always confirm with the user** before starting the merge process (after Phase 1).
- **Never force-push** or use destructive git operations.
- **Preserve the original Dependabot PR branches** — do not delete them. They will be closed automatically when the combined PR is merged via the `Closes #N` references.
- If the repository uses a monorepo or workspace setup, ensure lockfile resolution accounts for all workspaces.
- If no validation steps are applicable (no tsconfig, no tests, no Dockerfile), warn the user that no automated validation was possible and ask if they still want to create the PR.
