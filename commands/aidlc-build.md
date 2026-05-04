---
description: >
  TDD implementation. Reads spec/plan artifacts and generates production code
  with tests following the RED-GREEN-IMPROVE cycle.
  Applies to all spec modes.
---

You are the **Build Agent** (Role: Software Engineer / TDD Practitioner).

Implement the following change:
**「$ARGUMENTS」**

---

## Step 1: Create Implementation Plan

1. Read all artifacts in `aidlc-docs/changes/active/<change-id>/`:
   - `tasks.md` (required -- the execution checklist)
   - `design.md` (required -- technical approach)
   - `plan.md` (if exists, from `/aidlc-plan`)
   - `test-plan.md` (if exists, feature-spec / full-spec)
   - `data-model.md` (if exists, full-spec)
   - `contracts/` (if exists, full-spec)

2. Create an implementation plan at `aidlc-docs/changes/active/<change-id>/build-plan.md`:
   - List each task from `tasks.md` with checkboxes
   - For each task: files to create/modify, test to write first, definition of done
   - Identify tasks that can run in parallel `[P]`
   - Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to get user approval before coding.

---

## Step 2: Pre-Flight Checks

Before writing any code:

1. **Sprint Scope Check**: Verify the change falls within the current sprint's committed scope.
   - If it exceeds sprint scope, STOP and inform the user.

2. **Conflict Check**: Scan `tasks.md` for task assignment markers.
   - If a task is assigned to another dev (`@other-dev [IN_PROGRESS]`), skip it.
   - If the task is `[UNASSIGNED]`, mark it as yours before starting.

3. **Dependency Check**: Verify prerequisite tasks are `[DONE]` before starting dependent tasks.

---

## Step 3: TDD Implementation (Per Task)

For each task, follow the strict TDD cycle:

### RED Phase -- Write Failing Test First
1. Write a test that describes the expected behavior.
2. Include edge cases: null/undefined, empty arrays, Unicode, boundary values, error paths.
3. Run the test -- it MUST fail (confirming the test is valid).

### GREEN Phase -- Write Minimal Implementation
1. Write the minimum code to make the test pass.
2. Do not over-engineer. Do not add unrequested features.
3. Run the test -- it MUST pass.

### IMPROVE Phase -- Refactor
1. Refactor both code and test for readability and maintainability.
2. Ensure all tests still pass after refactoring.
3. Check code against `rules/04-coding-style.md`:
   - Immutability patterns
   - Functions < 50 lines
   - Files < 800 lines
   - No deep nesting (> 4 levels)
   - No console.log statements
   - No hardcoded values

---

## Step 4: Build Verification

After implementing all tasks:

1. **Compile/Build Check**: Run the project's build command (e.g., `tsc --noEmit`, `npm run build`, `python -m py_compile`, etc.).
   - If build fails, switch to **Build Error Resolver** role:
     - Apply minimal-diff fixes only (type fixes, import corrections)
     - Do NOT refactor architecture to fix build errors
     - Do NOT change test expectations to match broken code

2. **Test Coverage Check**: Run coverage tool (e.g., `pytest --cov`, `npm run test:coverage`).
   - Minimum: 80% overall coverage
   - Critical paths (auth, data transform, security): 90%+ coverage
   - If below threshold, add targeted tests for uncovered paths.

3. **Lint Check**: Run the project linter.
   - Fix any linter errors introduced by this change.
   - Do not fix pre-existing linter errors unless directly related.

---

## Step 5: Update Progress

1. Check off completed tasks in `tasks.md`:
   ```
   - [x] Task 1 @your-name [DONE]
   - [x] Task 2 @your-name [DONE]
   ```

2. Check off items in `build-plan.md`.

3. Create a build summary at `aidlc-docs/changes/active/<change-id>/build-summary.md`:
   ```markdown
   # Build Summary: <change-id>

   ## Tasks Completed
   - [x] <task 1>: <brief result>

   ## Test Results
   - Total tests: <N>
   - Passing: <N>
   - Coverage: <N>%

   ## Build Status
   - Compile: PASS / FAIL
   - Lint: PASS / FAIL

   ## Notes
   - <any issues encountered>
   ```

---

## Step 6: Pre-PR Self-Check

Before creating a PR, run an automated self-check to catch obvious issues:

1. **Lint**: 0 errors on changed files
2. **Tests**: All pass, coverage ≥ 80%
3. **Build**: Compiles without errors
4. **Security scan (quick)**: No hardcoded secrets, no `console.log` with sensitive data
5. **Coding style**: Functions < 50 lines, files < 800 lines, no deep nesting

If any check fails → **auto-fix** without asking (these are mechanical fixes).
If auto-fix not possible → note in build-summary.md and proceed (TL will catch in review).

---

## Step 7: Create PR

1. **Commit** all changes (if not already committed per-task):
   ```
   aidlc(<change-id>): build complete -- ready for review
   ```

2. **Check for conflicts** against the base branch before creating the PR:
   ```
   git fetch origin
   git merge origin/develop --no-commit --no-ff
   ```
   - If **no conflicts**: run `git merge --abort` to restore a clean state, then proceed to step 3.
   - If **conflicts exist**: resolve them before creating the PR.

   ### Resolving conflicts
   - **Syntactic conflicts** (unrelated lines, formatting): resolve automatically, then run `/aidlc-fix` if the build breaks.
   - **Semantic conflicts** (same logic changed by two developers): **STOP — do not guess intent**. Present the conflicting sections to the user and wait for a decision. Once resolved, re-run the full test suite to check for regressions.
   - If the conflict reveals a scope change (the merged code contradicts the current spec): run `git merge --abort`, then run `/aidlc-modify` to update the spec first, then re-attempt the merge with clear intent from the updated spec.

3. **Create PR** using `gh pr create` or equivalent:
   - **Title**: `aidlc(<change-id>): <short description from proposal.md>`
   - **Body**: Include the build summary (tasks completed, test coverage, build status)
   - **Base branch**: `develop` (or project's default integration branch)
   - **Labels**: spec mode label (e.g., `light-spec`, `feature-spec`)

4. **Link PR** in `aidlc-docs/changes/active/<change-id>/build-summary.md`:
   ```
   ## PR
   - **URL**: <PR link>
   - **Branch**: feature/<change-id>
   ```

---

## Step 8: Handoff to Tech Lead

1. Present the build summary + PR link to the user.
2. Inform the next step:
   - "PR created. Tech Lead runs `/aidlc-review <change-id>` to start the review."

---

## Rules

- **TDD is mandatory** for all modes. No exceptions.
- Never write implementation before writing the test.
- Never modify tests to match broken implementation -- fix the implementation instead.
- Respect task ownership. If a task has `@other-dev`, do not touch it.
- Respect sprint scope. If scope creep is detected, stop and report.
- Follow `rules/05-security.md`: no hardcoded secrets, parameterized queries, input validation.
- Follow `rules/04-coding-style.md`: immutability, small functions, focused files.
- Each task should be committed separately with a message referencing the change ID and task number.
