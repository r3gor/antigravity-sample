---
description: Reviews staged git changes against project rules and best practices, provides actionable feedback, and assists in crafting the commit message.
---

# Pre-Commit Review (/pre-commit-review)

This workflow acts as a senior code reviewer and git assistant. It inspects all staged changes, validates them against the active workspace rules and skills, provides structured feedback, and finally helps craft a clean commit message.

## Instructions for the Agent:

When the user invokes this workflow (e.g. `/pre-commit-review`), strictly follow these steps in order:

### Step 0: Branch Guard 🚨
- Run `git branch --show-current` to detect the active branch.
- If the current branch is `main` or `master`, **immediately stop the workflow** and respond with:

  > 🚫 **Direct commits to `main`/`master` are not allowed through this workflow.**
  > Committing directly to a protected branch is a risky practice that bypasses code review and CI/CD pipelines.
  >
  > **Recommended actions:**
  > - Create a feature branch: `git checkout -b feat/<your-feature-name>`
  > - Or a fix branch: `git checkout -b fix/<your-fix-name>`
  > - Stage your changes there and re-run `/pre-commit-review`
  >
  > *(If you intentionally need to commit to `main`, you can do so manually from the terminal.)*

- If the current branch is **any other branch**, display a brief confirmation:
  > ✅ Branch: `<branch-name>` — safe to proceed.
  Then continue to Step 1.

### Step 1: Capture Staged Changes
- Run `git diff --staged` to get the full diff of all staged files.
- If **nothing is staged**, run `git diff HEAD` to show unstaged changes and warn the user:
  > ⚠️ No staged changes found. Showing unstaged changes instead. Run `git add <files>` to stage before committing.
- If **no changes at all**, stop and inform the user there is nothing to review.

### Step 2: Understand the Context
- Identify the programming language(s) and framework(s) involved in the changes.
- List all files modified, added, or deleted.
- Note whether this touches production code, tests, configuration, or documentation.

### Step 3: Review Against Workspace Rules
Apply each of the following checks to ALL changed files:

- **Module Import Boundaries** (`module-imports` rule):
  - Are internal imports using relative paths (`./`, `../`)?
  - Are cross-module imports using path aliases (`@/`)?
  - Are barrel files (`index.ts`) used only at the module root?
  - Is there any "path hell" (e.g., `../../../../`)?

- **Avoid Reinventing the Wheel** (`avoid-reinvent-wheel` rule, if present):
  - Is any new utility or helper logic duplicating functionality that already exists in the codebase or a well-known library?

### Step 4: Review Against Skills & Architecture
If relevant skills are active (e.g., `hexagonal-pragmatic-architecture`, `solid-architectural-governance`), apply the following:

- **SOLID Principles**: Flag any clear Single Responsibility, Open/Closed, or Dependency Inversion violations in the changed code.
- **Layer Boundaries**: Does any change cause a domain layer to depend on an infrastructure or framework concern?
- **Code Smells**: Identify deeply nested logic, magic numbers/strings, overly large functions, or duplicated logic blocks.
- **Test Coverage**: If production code was changed, check if corresponding test files were also modified. If not, flag it as a warning.

- **Naming Analysis** (`naming-analyzer` skill): Apply the naming analyzer to all changed files and flag:
  - 🔴 **Misleading names** — function/method name doesn't match its actual behavior (e.g., a `get*` function that also writes to DB).
  - 🟡 **Vague or generic names** — `data`, `info`, `temp`, `process`, `x`, single-letter variables outside loops.
  - 🟡 **Boolean naming** — booleans missing `is`, `has`, `can`, or `should` prefix (e.g., `user.active` → `user.isActive`).
  - 🟡 **Unexplained abbreviations** — names that obscure meaning (e.g., `usrCfg`, `calcTtl`). Well-known exceptions (`id`, `url`, `api`, `html`) are acceptable.
  - 🟡 **Magic numbers/strings** — raw literals that should be named constants (e.g., `3600000` → `ONE_HOUR_IN_MS`).
  - 🟡 **Convention inconsistencies** — mixing `camelCase` and `snake_case` within the same language context.


### Step 5: Generate the Review Report
Present a structured Markdown report with these sections:

- 🔴 **Blockers** — Issues that should be fixed before committing (rule violations, broken architecture boundaries, etc.).
- 🟡 **Warnings** — Non-blocking issues that are worth addressing soon (code smells, missing tests, improvement suggestions).
- 🟢 **Looks Good** — Briefly highlight what was done well to reinforce good practices.

For every 🔴 Blocker, **always include a concrete before/after code snippet** showing the problem and the fix.

> **Pause here.** Ask the user: *"Would you like me to automatically fix any of the blockers before we proceed to the commit?"*
> - If YES → Apply the fixes, show a summary of changes made, then continue to Step 6.
> - If NO → Continue to Step 6 without changes.

### Step 6: Craft the Commit Message
Once the user is satisfied with the changes, generate a commit message following the **Conventional Commits** standard (`https://www.conventionalcommits.org`):

```
<type>(<scope>): <short imperative description>

[optional body: explain the WHY, not the WHAT]

[optional footer: BREAKING CHANGE, closes #issue]
```

**Allowed types:** `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `style`, `ci`, `build`

Rules for the message:
- The subject line must be **50 characters or fewer**.
- Use the **imperative mood** in the subject (e.g., "add", "fix", "update" — not "added" or "fixes").
- Infer the `scope` from the primary module or folder touched (e.g., `feat(user)`, `fix(auth)`).
- If the diff touches multiple unrelated concerns, **warn the user** that they might want to split the commit for clarity.

Present the suggested message and ask the user to confirm or request adjustments.

### Step 7: Commit (with user approval)
- Once the user approves the message, run: `git commit -m "<message>"`.
- Confirm the commit was successful and show the resulting commit hash.
- Ask if they'd also like to `git push` now.
