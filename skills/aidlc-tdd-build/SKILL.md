---
name: aidlc-tdd-build
description: >-
  Implement code following strict TDD (RED-GREEN-IMPROVE) cycle per AIDLC Custom standards.
  Use when running /aidlc-build, implementing a task from tasks.md, or when the user asks
  to build/implement a feature with tests.
---

# AIDLC TDD Build

## When to Use

- User runs `/aidlc-build` or asks to implement a task
- User asks to "build", "implement", or "code" a feature with tests
- User wants TDD-style implementation

## TDD Cycle (Per Task)

For each task in `tasks.md`, execute strictly in order:

### 1. RED -- Write Failing Test First

```
1. Read the task's definition of done and acceptance criteria
2. Write a test that describes the expected behavior
3. Include edge cases: null/undefined, empty, Unicode, boundary values, error paths
4. Run the test → it MUST FAIL
   - If it passes → test is wrong (testing existing behavior, not new)
   - Fix the test until it fails for the right reason
```

### 2. GREEN -- Minimal Implementation

```
1. Write the MINIMUM code to make the test pass
2. Do NOT:
   - Over-engineer
   - Add features not in the spec
   - Optimize prematurely
   - Add "nice to have" code
3. Run the test → it MUST PASS
```

### 3. IMPROVE -- Refactor

```
1. Refactor both code AND test for readability
2. Check against coding standards:
   - Functions < 50 lines
   - Files < 800 lines
   - No deep nesting (> 4 levels)
   - Immutable patterns (new objects, not mutations)
   - No console.log / hardcoded values
3. Run ALL tests → they MUST still PASS
```

## Pre-Flight Checks

Before writing any code:

1. **Read spec artifacts**: `design.md`, `tasks.md`, `test-plan.md` (if exists)
2. **Sprint scope check**: Is this task in the current sprint?
3. **Conflict check**: Is the task `[UNASSIGNED]`? Mark `@your-name [IN_PROGRESS]`
4. **Dependency check**: Are prerequisite tasks `[DONE]`?
5. **DEP check**: Any `[DEP: Task X.Y]`? → Verify interface contract is agreed

## Coverage Targets

| Scope | Minimum |
|-------|---------|
| Overall | 80% |
| Auth, security, data transform | 90% |

If below threshold → add targeted tests for uncovered paths before proceeding.

## Build Verification

After all tasks complete:

1. Run full build (`tsc --noEmit`, `npm run build`, `python -m py_compile`, etc.)
2. Run full test suite with coverage
3. If build fails → apply minimal-diff fixes only (do NOT refactor architecture)
4. Generate `build-summary.md` with: tasks completed, coverage, test results

## Commit Convention

After each task passes:

```
aidlc(CHG-xxx): task N.M complete -- <short description>
```

After all tasks pass:

```
aidlc(CHG-xxx): build complete -- ready for review
```
