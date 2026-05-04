---
description: >
  Tech Lead's PR review command. Runs TDD Expert, Code Reviewer, and Security Reviewer
  in parallel on an existing PR. Produces a consolidated review report with per-issue decisions.
  Maps to "Review PR & Decision" (Step ⑥) in the 3-Lane Workflow.
  Prerequisite: PR must exist (created by /aidlc-build).
---

You are the **Review Agent**, acting on behalf of the **Tech Lead**. You orchestrate three expert roles in parallel to review the PR.

This command is the **Tech Lead's official review gate**. The Developer's self-check happens in Step 6 of `/aidlc-build` (lint, tests, build, security scan) — not here.

Prerequisite: PR must exist (created by `/aidlc-build`).

Review the PR for the following change:
**「$ARGUMENTS」**

---

## Step 1: Load Context

1. Read `aidlc-docs/changes/active/<change-id>/build-summary.md` for build results and **PR link**.
2. If PR link exists, read the PR diff (`gh pr diff <number>` or `git diff develop...HEAD`).
3. Read the spec artifacts (design.md, test-plan.md, acceptance.md) for review criteria.
4. Identify changed files by reading `tasks.md` (completed tasks list file paths).
5. If no PR exists, inform user: "No PR found. Developer should run `/aidlc-build` first to create the PR."

---

## Step 2: Run 3 Expert Reviews in Parallel

**IMPORTANT: Complete ALL three reviews fully before presenting anything to the user.
Do NOT stop mid-review to ask about individual findings.
Do NOT offer to fix issues during the review phase.
Collect ALL findings first → consolidate → present the full report → THEN ask for decision.**

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

**IMPORTANT: Do NOT present partial results. Wait until ALL 3 experts have finished,
then write the COMPLETE report before moving to Step 4.**

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

## Step 4: Present Full Report

**CRITICAL: Present the COMPLETE review report FIRST. Do NOT stop mid-review to ask about individual findings.**

1. Present the full review report (Summary + all 3 expert findings).

2. After the full report, present ALL findings in a numbered list with each issue's recommendation:

```
Issues found:

CRITICAL (must fix before merge):
  C-1: [Code] completeSection() called in render body → wrap in useEffect
       File: AttributeCharts.tsx:42
  C-2: [Security] API endpoint missing auth middleware
       File: api/export.py:15

WARNING (should fix):
  W-1: [Code] Function exportData() is 85 lines → split into smaller functions
       File: api/export.py:30

SUGGESTION (nice to have):
  S-1: [TDD] Add test for concurrent reset password requests
```

---

## Step 5: Per-Issue Decision

**Ask the user to decide on EACH issue individually.** Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` — one question per issue, with these options:
- **Fix**: Agree with finding, fix as recommended
- **Fix differently**: Agree it's an issue, but propose a different approach (user explains)
- **Dismiss**: Not a real issue / acceptable risk (user explains reason)
- **Defer**: Valid issue but fix in a future task, not now

If the user selects **Fix differently** or **Dismiss**, ask for their explanation before proceeding.

---

## Step 6: Execute & Final Decision

1. **Apply fixes** for all issues marked "Fix" or "Fix differently" (using user's approach).
2. **Log dismissed issues** in review-report.md with the user's reason.
3. **Log deferred issues** as TODOs for future tasks.

4. After fixes are applied, present a summary:

```
Fixed:    C-1 (as recommended), W-1 (as recommended)
Deferred: S-1 (→ future task)
Dismissed: C-2 (user reason: "auth handled by API gateway, not middleware")
```

5. Ask for the **final PR decision** using the platform confirmation mechanism defined in `rules/09-platform-confirmation.md`:
   - **Accept → Merge**: All critical issues resolved. Merge to develop.
   - **Re-review**: Run `/aidlc-review` again to verify fixes.
   - **Postpone**: Return to PM for rescheduling.
   - **Reject**: Fundamental flaw, return to `/aidlc-spec` or `/aidlc-plan`.

6. Based on decision:
   - **Accept**: "Review passed. PR is approved. Run `/aidlc-release <change-id>` to merge & release."
   - **Re-review**: Apply fixes, then re-run the full review on the updated code.
   - **Postpone**: Note reason in review-report.md, keep PR open.
   - **Reject**: Close PR with rejection reason, archive spec package.

---

## Rules

- All 3 expert reviews run in parallel for efficiency.
- **Complete ALL reviews before presenting to user. Never stop mid-review to ask questions.**
- **Present the full report first, then ask per-issue decisions. Never offer to fix issues during review.**
- CRITICAL findings from ANY expert block the merge (unless user explicitly dismisses with reason).
- If user dismisses a CRITICAL finding, log the reason prominently in review-report.md.
- Do NOT auto-pass. Even if everything looks clean, present the report and wait for human decision.
- The review report becomes a permanent artifact in the spec package.
- For light-spec with clean findings, the report can be abbreviated (skip per-issue if 0 CRITICAL + 0 WARNING).
- If the review reveals a spec flaw (not just code), recommend going back to `/aidlc-confirm` with updated spec.
- Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` for all decision prompts.
