---
description: Execute an implementation plan step-by-step, following TDD principles in an Ash Framework codebase.
---

# Ash Execute

Execute a plan created by `ash-plan`. This skill implements changes methodically, writing tests first, then implementation, validating at each step.

---

## Prerequisites

Before executing, ensure you have:

1. **A valid implementation plan** - Created via `ash-plan` with:
   - Summary of the change
   - List of affected files
   - Ordered implementation steps
   - Test cases defined
2. **User approval** - The plan has been reviewed and approved by the user.

If no plan exists, stop and use `ash-plan` first.

---

## Phase 1: Setup

Prepare for implementation.

1. **Review the plan** - Re-read the plan to understand the full scope.
2. **Check current state** - Ensure the codebase is in a clean state:
   - No uncommitted changes that would conflict
   - Tests currently pass (`mix test`)
3. **Identify the first test** - Locate the first test case from the plan.

---

## Phase 2: Test-First Implementation

Follow this cycle for each feature/change in the plan:

### Step A: Write the Test

1. **Create or locate the test file** - Follow existing test directory structure.
2. **Write the test** - Implement the test case exactly as specified in the plan.
3. **Run the test** - Verify it **fails** for the right reason.
   \`\`\`bash
   mix test path/to/test_file.exs:LINE --trace
   \`\`\`
4. **Confirm failure** - The test should fail because the feature doesn't exist yet, not due to syntax errors or setup issues.

### Step B: Implement the Feature

1. **Write minimal code** - Implement just enough to make the test pass.
2. **Follow Ash patterns** - Use Ash idioms (actions, resources, calculations) over pure Elixir.
3. **Check usage rules** - If unsure about a package's API, consult `usage-rules` or `hex-docs-search`.
4. **Run the test again** - Verify it now **passes**.
   \`\`\`bash
   mix test path/to/test_file.exs:LINE --trace
   \`\`\`

### Step C: Refactor (if needed)

1. **Clean up** - Improve code quality without changing behavior.
2. **Run all related tests** - Ensure nothing broke.
   \`\`\`bash
   mix test path/to/test_file.exs --trace
   \`\`\`

### Step D: Repeat

Move to the next test case in the plan. Repeat Steps A-C until all test cases pass.

---

## Phase 3: Validation

After all implementation steps are complete:

1. **Run the full test suite**
   \`\`\`bash
   mix test
   \`\`\`
2. **Fix any regressions** - If other tests broke, investigate and fix before proceeding.
3. **Check compilation warnings**
   \`\`\`bash
   mix compile --warnings-as-errors
   \`\`\`
4. **Run static analysis** (if available)
   \`\`\`bash
   mix credo --strict
   mix dialyzer
   \`\`\`

---

## Phase 4: Completion

1. **Review all changes** - Summarize what was implemented.
2. **Mark plan as complete** - Update the implementation steps to `[x]`.
3. **Report to user** - Provide a summary:
   - Files created/modified
   - Tests added/passing
   - Any deviations from the plan and why

---

## Execution Rules

### One Step at a Time

- Complete each implementation step fully before moving to the next.
- Do not skip ahead or parallelize unrelated changes.
- If a step reveals the plan needs adjustment, pause and discuss with the user.

### Test Discipline

- **Never** write implementation code before the corresponding test.
- **Never** mark a step complete if its test is failing.
- **Never** comment out or skip failing tests to proceed.

### Ash Framework Compliance

Always prefer Ash constructs:

| Instead of...               | Use...                          |
|-----------------------------|----------------------------------|
| Raw Ecto queries            | Ash actions (`read`, `create`)  |
| Custom validation functions | Ash validations/changes         |
| Manual authorization checks | Ash policies                    |
| Computed fields in queries  | Ash calculations                |
| Standalone GenServers       | Ash resources with actions      |

### Error Handling

If something goes wrong during execution:

1. **Stop immediately** - Do not continue with a broken state.
2. **Diagnose the issue** - Read error messages carefully.
3. **Check documentation** - Use `usage-rules` or `hex-docs-search` for guidance.
4. **Consult the user** - If the issue requires a plan change, discuss before proceeding.

### Deviations from Plan

If during execution you discover:
- The plan has a mistake
- A better approach exists
- Additional steps are needed

**Do not silently deviate.** Inform the user and get approval before changing course.

---

## Progress Reporting

After completing each major step, report progress:

\`\`\`markdown
✅ Step 1: Created test for user registration
✅ Step 2: Implemented User.register action
⏳ Step 3: Create test for email validation (in progress)
⬚ Step 4: Implement email validation change
⬚ Step 5: Run full test suite
\`\`\`

This keeps the user informed and creates a clear audit trail.