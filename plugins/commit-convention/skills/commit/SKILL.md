---
description: Use this skill whenever creating a git commit. Enforces conventional commit format with short, precise messages and no co-author lines.
---

# Commit Convention

This skill defines the commit message format to use for EVERY commit. Follow these rules strictly.

---

## Format

```
<type>: <short description>
```

- **One line only.** No body, no footer, no blank lines — unless the user explicitly asks for a multi-line message.
- **Lowercase** everything after the type prefix.
- **Max 72 characters** total.
- **Imperative mood** — "add feature" not "added feature" or "adds feature".

---

## Allowed Types

| Type       | When to use                                      |
|------------|--------------------------------------------------|
| `feat`     | New feature or capability                        |
| `fix`      | Bug fix                                          |
| `refactor` | Code restructuring without behavior change       |
| `chore`    | Maintenance, dependencies, tooling, config       |
| `docs`     | Documentation only                               |
| `style`    | Formatting, whitespace, semicolons (no logic)    |
| `test`     | Adding or updating tests                         |
| `perf`     | Performance improvement                          |
| `ci`       | CI/CD pipeline changes                           |
| `build`    | Build system or external dependency changes      |
| `revert`   | Reverting a previous commit                      |

If a commit spans multiple types, pick the **primary** one.

---

## Optional Scope

A scope may be added in parentheses when it adds clarity:

```
feat(auth): add token refresh endpoint
fix(parser): handle empty input gracefully
```

Only use a scope when it genuinely helps — don't force one.

---

## Strict Rules

1. **NEVER add a `Co-Authored-By` line.** Do not mention Claude, Claude Code, Anthropic, AI, or any co-author attribution. The commit must look like it was written entirely by the user.
2. **Keep it short and precise.** Say what changed, not how or why — the diff shows that.
3. **No periods** at the end of the description.
4. **No capitalization** of the first word after the type prefix.

---

## Examples

Good:
```
feat: add user avatar upload
fix: prevent duplicate webhook events
refactor: extract validation into shared module
chore: update eslint to v9
test: add coverage for edge cases in parser
docs: update API authentication guide
```

Bad:
```
feat: Add user avatar upload.          # capitalized, has period
update stuff                           # no type prefix, vague
fix: this fixes the bug where...       # too verbose
feat: add feature                      # too vague
```

---

## Staging

When staging files for commit:
- **Be specific** — stage only the files relevant to the change.
- **Never** use `git add -A` or `git add .` unless every changed file belongs in the commit.
- If unrelated changes exist, ask the user whether to include or exclude them.
