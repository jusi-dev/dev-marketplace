---
description: Run, analyze, and debug tests in an Ash Framework codebase with intelligent failure diagnosis.
---

# Ash Test

Run tests, interpret results, and diagnose failures. This skill helps understand test output, identify root causes, and suggest fixes—all within the Ash Framework context.

---

## When to Use This Skill

- Running specific tests or the full test suite
- Diagnosing why a test is failing
- Understanding test output and error messages
- Verifying that changes haven't broken existing functionality

---

## Phase 1: Running Tests

### Run Specific Test

Target a single test by file and line number:

\`\`\`bash
mix test path/to/test_file.exs:LINE --trace
\`\`\`

### Run Test File

Run all tests in a file:

\`\`\`bash
mix test path/to/test_file.exs --trace
\`\`\`

### Run Tests by Tag

Run tests with a specific tag:

\`\`\`bash
mix test --only tag_name
\`\`\`

### Run Full Suite

Run all tests:

\`\`\`bash
mix test
\`\`\`

### Useful Flags

| Flag | Purpose |
|------|---------|
| `--trace` | Show detailed test execution |
| `--seed 0` | Disable randomization for reproducible order |
| `--max-failures 1` | Stop after first failure |
| `--stale` | Only run tests affected by recent changes |
| `--failed` | Re-run only previously failed tests |

---

## Phase 2: Interpreting Results

### Success Output

\`\`\`
....

Finished in 0.1 seconds (0.05s async, 0.05s sync)
4 tests, 0 failures
\`\`\`

All tests passed. Proceed with confidence.

### Failure Output

\`\`\`
  1) test creates a user with valid attributes (MyApp.Accounts.UserTest)
     test/my_app/accounts/user_test.exs:15
     Assertion with == failed
     code:  assert result.email == "test@example.com"
     left:  nil
     right: "test@example.com"
     stacktrace:
       test/my_app/accounts/user_test.exs:20: (test)
\`\`\`

Key information to extract:
1. **Test name and location** - Which test failed and where
2. **Assertion type** - What kind of check failed
3. **Left vs Right** - Actual value vs expected value
4. **Stacktrace** - Where in the code the failure occurred

---

## Phase 3: Diagnosing Failures

Follow this systematic approach:

### Step 1: Categorize the Failure

| Error Type | Typical Cause |
|------------|---------------|
| `** (UndefinedFunctionError)` | Missing function, wrong arity, or module not loaded |
| `** (KeyError)` | Accessing a key that doesn't exist in a map/struct |
| `** (Ash.Error.Invalid)` | Ash validation or changeset error |
| `** (Ash.Error.Forbidden)` | Ash policy authorization failure |
| `** (Ecto.NoResultsError)` | Query expected a result but found none |
| `** (MatchError)` | Pattern match failed |
| `Assertion with == failed` | Value doesn't match expectation |

### Step 2: Ash-Specific Errors

For Ash errors, dig deeper:

#### Validation Errors
\`\`\`elixir
** (Ash.Error.Invalid) Invalid Error
* email: must be present
\`\`\`
**Cause:** Required attribute missing or validation failed.
**Check:** Are all required attributes provided? Do values pass validations?

#### Policy Errors
\`\`\`elixir
** (Ash.Error.Forbidden) Forbidden Error
forbidden on MyApp.Accounts.User.create
\`\`\`
**Cause:** Authorization policy denied the action.
**Check:** Is the actor set? Does the policy allow this action for this actor?

#### Action Errors
\`\`\`elixir
** (Ash.Error.Invalid) Invalid Error
No such action :register for MyApp.Accounts.User
\`\`\`
**Cause:** Calling an action that doesn't exist.
**Check:** Is the action defined on the resource? Is it spelled correctly?

### Step 3: Inspect the Test Setup

Common setup issues:

1. **Missing fixtures** - Data the test depends on wasn't created
2. **Wrong actor** - Test uses wrong user or no user for authorization
3. **Database state** - Previous test left data that affects this test
4. **Async conflicts** - Async tests sharing mutable state

### Step 4: Check Recent Changes

If a previously passing test now fails:

1. What code changed since it last passed?
2. Did a dependency update?
3. Did the test data/fixtures change?

---

## Phase 4: Common Fixes

### Ash Validation Failed

\`\`\`elixir
# Problem: Required field missing
User
|> Ash.Changeset.for_create(:create, %{name: "Test"})
|> Ash.create!()

# Fix: Provide all required fields
User
|> Ash.Changeset.for_create(:create, %{name: "Test", email: "test@example.com"})
|> Ash.create!()
\`\`\`

### Ash Policy Forbidden

\`\`\`elixir
# Problem: No actor provided
User
|> Ash.Changeset.for_create(:create, params)
|> Ash.create!()

# Fix: Provide actor or use authorize?: false for tests
User
|> Ash.Changeset.for_create(:create, params)
|> Ash.create!(actor: current_user)

# Or for seed data / setup:
User
|> Ash.Changeset.for_create(:create, params)
|> Ash.create!(authorize?: false)
\`\`\`

### Relationship Not Loaded

\`\`\`elixir
# Problem: Accessing unloaded relationship
user = Ash.get!(User, id)
user.posts  # => #Ash.NotLoaded<...>

# Fix: Load the relationship
user = Ash.get!(User, id, load: [:posts])
user.posts  # => [%Post{}, ...]
\`\`\`

### Stale Data in Test

\`\`\`elixir
# Problem: Variable holds old data
user = create_user()
update_user(user, %{name: "New Name"})
assert user.name == "New Name"  # Fails! user still has old name

# Fix: Reload the data
user = create_user()
update_user(user, %{name: "New Name"})
user = Ash.get!(User, user.id)
assert user.name == "New Name"
\`\`\`

---

## Phase 5: Reporting Results

After diagnosing, report clearly:

### For Passing Tests
\`\`\`markdown
✅ All tests passed (15 tests, 0 failures)
\`\`\`

### For Failing Tests
\`\`\`markdown
❌ Test failed: `test creates a user with valid attributes`

**Location:** test/my_app/accounts/user_test.exs:15

**Error:** Ash.Error.Invalid - email: must be present

**Root Cause:** The test is not providing the `email` attribute which is required by the User resource.

**Suggested Fix:**
Change line 17 from:
`Ash.Changeset.for_create(:create, %{name: "Test"})`
To:
`Ash.Changeset.for_create(:create, %{name: "Test", email: "test@example.com"})`
\`\`\`

---

## Test Writing Guidelines

When creating new tests (via `ash-plan` and `ash-execute`), follow these patterns:

### Structure

\`\`\`elixir
defmodule MyApp.Accounts.UserTest do
  use MyApp.DataCase, async: true

  alias MyApp.Accounts.User

  describe "create action" do
    test "creates user with valid attributes" do
      # Arrange
      params = %{name: "Test", email: "test@example.com"}
      
      # Act
      result = User
               |> Ash.Changeset.for_create(:create, params)
               |> Ash.create!()
      
      # Assert
      assert result.name == "Test"
      assert result.email == "test@example.com"
    end

    test "returns error with invalid email" do
      params = %{name: "Test", email: "invalid"}
      
      assert {:error, %Ash.Error.Invalid{}} =
               User
               |> Ash.Changeset.for_create(:create, params)
               |> Ash.create()
    end
  end
end
\`\`\`

### Best Practices

- **Use `async: true`** when tests don't share mutable state
- **Use descriptive test names** that explain what's being tested
- **Follow Arrange-Act-Assert** pattern for clarity
- **Test both success and failure** cases
- **Use `!` functions** (like `create!`) when failure should crash the test
- **Use non-bang functions** when testing error handling