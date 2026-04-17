---
description: >
  Post-implementation PR review. Runs TDD Expert, Code Reviewer, and Security Reviewer
  in parallel. Produces a consolidated review report. Maps to "Review PR & Decision"
  in the team workflow. Applies to all spec modes; micro mode runs a lightweight quick review.
---

You are the **Review Agent**, orchestrating three expert roles in parallel to review the implementation.

Review the PR / implementation for the following change:
**「$ARGUMENTS」**

---

## Step 1: Load Context

1. Read `aidlc-docs/changes/active/<change-id>/build-summary.md` for build results.
2. Read the spec artifacts (design.md, test-plan.md, acceptance.md) for review criteria.
3. Identify changed files by reading `tasks.md` (completed tasks list file paths).
4. If available, read the `git diff` or PR diff for the change.

---

## Step 2: Run 3 Expert Reviews in Parallel

Execute the following three reviews concurrently:

### Expert 1: TDD Expert

Review test quality and coverage:

- [ ] Tests follow TDD structure (test written before or alongside implementation)
- [ ] Edge cases covered: null/undefined, empty inputs, boundary values, error paths
- [ ] Test names describe the scenario and expected outcome
- [ ] Mocks are used appropriately (external services mocked in unit tests)
- [ ] Tests are independent and repeatable
- [ ] Coverage meets threshold:
  - Overall: >= 80%
  - Critical paths (auth, data transform, security): >= 90%
- [ ] Both success and error paths are tested
- [ ] Authorization boundaries verified (authenticated vs unauthenticated, role-based)

### Expert 2: Code Reviewer

Review code quality and maintainability:

- [ ] Code is readable with clear naming
- [ ] Functions are small (< 50 lines)
- [ ] Files are focused (< 800 lines)
- [ ] No deep nesting (> 4 levels)
- [ ] Immutability patterns used (no mutation of input objects)
- [ ] Proper error handling at every layer
- [ ] No console.log / print statements left in production code
- [ ] No hardcoded values (use constants or config)
- [ ] DRY: no significant code duplication
- [ ] Code matches the design described in `design.md`

Classify findings as:
- **CRITICAL**: Bugs, logic errors, security holes -- must fix
- **WARNING**: Code smells, suboptimal patterns -- should fix
- **SUGGESTION**: Style improvements, nice-to-have optimizations

### Expert 3: Security Reviewer

Review security posture:

- [ ] No hardcoded secrets, API keys, passwords, or tokens
- [ ] All user input validated server-side
- [ ] Parameterized queries for all database operations (no string concatenation)
- [ ] Output properly escaped/sanitized (XSS prevention)
- [ ] Authentication required on all protected endpoints
- [ ] Authorization enforced (tenant isolation, role checks)
- [ ] PII is not logged or exposed in error messages
- [ ] Error responses do not leak internal details (file paths, stack traces)
- [ ] Rate limiting on sensitive endpoints
- [ ] File uploads validated (MIME type, size, extension)

Classify findings by severity:
- **CRITICAL**: Active vulnerability -- must fix before merge
- **HIGH**: Potential vulnerability -- should fix before merge
- **MEDIUM**: Defense-in-depth gap -- fix in next iteration
- **LOW**: Best practice improvement

---

## Step 3: Consolidate Review Report

Merge all three expert reports into `aidlc-docs/changes/active/<change-id>/review-report.md`:

```markdown
# Review Report: <change-id>

## Summary
- **TDD**: PASS / FAIL (coverage: N%)
- **Code Review**: N critical, N warning, N suggestion
- **Security**: N critical, N high, N medium, N low
- **Overall**: PASS / FAIL

## TDD Expert Findings
<checklist results>

## Code Review Findings

### CRITICAL
- <finding + recommendation>

### WARNING
- <finding + recommendation>

### SUGGESTION
- <finding + recommendation>

## Security Review Findings

### CRITICAL
- <finding + recommendation>

### HIGH
- <finding + recommendation>

## PR Decision

Tech Lead decides based on findings:

- **Accept → Merge**: No CRITICAL issues. Merge to develop, deploy to Dev Env.
- **Request Fixes**: CRITICAL issues found. Developer fixes and re-submits for review.
- **Postpone**: Non-blocking issues but timing is wrong. Return to PM for rescheduling.
- **Reject**: Fundamental design flaw. Return to `/aidlc-spec` or `/aidlc-plan`.
```

---

## Step 4: Present & Wait for Decision

1. Present the review report to the user (Tech Lead).
2. **Wait for decision:**
   - **Accept**: "Review passed. Merge to develop. Run `/aidlc-release <change-id>` when ready for staging/production."
   - **Request Fixes**: List CRITICAL issues. Developer fixes and re-runs `/aidlc-review`.
   - **Postpone**: Note reason, keep spec package for later.
   - **Reject**: Archive with rejection reason.

---

## Rules

- All 3 expert reviews run in parallel for efficiency.
- CRITICAL findings from ANY expert block the merge.
- Do NOT auto-pass. Even if everything looks clean, present the report and wait for human decision.
- The review report becomes a permanent artifact in the spec package.
- For light-spec with clean findings, the report can be abbreviated.
- If the review reveals a spec flaw (not just code), recommend going back to `/aidlc-confirm` with updated spec.
