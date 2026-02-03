---
description: Create a detailed implementation plan for features, bug fixes, or code changes in an Ash Framework codebase.
---

# Ash Plan

Create a structured, actionable plan before implementing any changes. The plan ensures consistency, follows TDD principles, and correctly uses the Ash Framework.

---

## Phase 1: Discovery

Understand what the user wants to achieve.

1. **Capture the request** - Summarize the user's initial input in your own words.
2. **Identify unknowns** - List any ambiguous requirements, edge cases, or missing details.
3. **Clarify with user** - Ask targeted questions before proceeding. Do not assume.

### When to ask for clarification

- The scope is unclear (e.g., "improve performance" - where? how much?)
- Multiple valid approaches exist and user preference matters
- Business rules or domain logic are involved
- The change affects external APIs or user-facing behavior

---

## Phase 2: Analysis

Investigate the codebase to ensure the plan fits existing patterns.

1. **Locate affected areas** - Identify files, modules, and resources that will be touched.
2. **Study existing patterns** - Examine similar implementations in the codebase:
   - Directory structure (where do similar files live?)
   - Module naming conventions
   - Resource structure (attributes, actions, calculations)
   - Test organization and style
3. **Gather usage rules** - Use the `usage-rules` skill to get best practices for affected packages. If no matching rule exists, fall back to `hex-docs-search`.
4. **Check for existing tests** - Determine if tests already cover the affected functionality.

---

## Phase 3: Planning

Produce the implementation plan.

### Output Format

Structure every plan with these sections:

```markdown
## Summary
One paragraph describing what will be implemented and why.

## Affected Files
- `path/to/file.ex` - Brief description of changes
- `path/to/new_file.ex` (new) - What this file will contain

## Implementation Steps
1. [ ] Step one - specific, actionable task
2. [ ] Step two - specific, actionable task
...

## Test Cases
- [ ] Test: description of what the test verifies
- [ ] Test: description of what the test verifies

## Dependencies
- Any packages to add/update
- Any migrations needed

## Open Questions (if any)
- Questions that arose during analysis
```

### Scope Guidelines

- **One plan = one logical change**. If a request involves multiple independent features, create separate plans.
- **Keep steps atomic**. Each step should be completable and testable on its own.
- **Tests come first**. Always list test cases before implementation steps (TDD).

---

## Key Principles

### Consistency First

Never introduce new patterns. Match existing codebase conventions for:
- Directory structure (tests in `test/`, resources in their domain folders)
- Resource structure (attributes → relationships → actions → calculations)
- Naming conventions (module names, function names, file names)
- Error handling patterns

### Test-Driven Development

Every change requires appropriate test coverage:
- Write test cases **before** implementation steps in the plan
- If modifying existing functionality, verify existing tests or add new ones
- Include both happy path and edge case tests

### Ash Framework First

The codebase uses Ash Framework. Always prefer Ash idioms over pure Elixir:
- Use Ash actions instead of raw Ecto queries
- Use Ash calculations instead of custom functions where applicable
- Use Ash policies for authorization
- Follow Ash resource patterns (not bare GenServers or custom modules)

### Correct Package Usage

LLMs may have outdated or incorrect knowledge about packages. Always:
1. Check `usage-rules` first for project-specific guidance
2. Fall back to `hex-docs-search` for official documentation
3. Prioritize documented usage over general LLM knowledge to avoid hallucinations