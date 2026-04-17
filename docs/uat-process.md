# UAT (User Acceptance Testing) Process

## When is UAT Required?

| Spec Mode | UAT Required? | Scope |
|-----------|--------------|-------|
| micro | No | N/A |
| light-spec | No (optional for visible UI changes) | N/A |
| feature-spec | Recommended | Key user flows |
| full-spec | **Required** | Full acceptance matrix |

## UAT Flow

```
/aidlc-review → Accept
        ↓
Deploy to Staging (pre-release/*)
        ↓
UAT Preparation (create test plan from acceptance.md)
        ↓
UAT Execution (PM/BrSE + Tech Lead + stakeholders)
        ↓
    ┌─── PASS → /aidlc-release → Deploy Production
    │
    └─── FAIL → Bug Report → Fix Loop
                    ↓
              Fix (micro/light-spec) → Re-deploy staging → Re-test
```

## Roles in UAT

| Role | Responsibility |
|------|---------------|
| **JP PM / BrSE** (Green Lane) | Lead UAT, validate against customer requirements, sign-off |
| **Tech Lead** (Blue Lane) | Validate technical quality, risk assessment, release readiness |
| **Dev Team** (Yellow Lane) | Fix bugs found during UAT, re-deploy |
| **Stakeholders** (if applicable) | Validate business requirements, provide feedback |

## UAT Checklist Template

Create at `aidlc-docs/changes/active/<change-id>/uat-checklist.md`:

```markdown
# UAT Checklist: <change-id>

## Metadata
- **Environment**: Staging (<URL>)
- **Version/Build**: <tag or commit hash>
- **Date**: YYYY-MM-DD
- **Testers**: <names>

## Pre-UAT Verification
- [ ] Staging environment is deployed and accessible
- [ ] Test data is prepared
- [ ] All automated tests pass on staging
- [ ] DB migration applied on staging
- [ ] Feature flags configured correctly

## Test Scenarios

### Scenario 1: <title from acceptance.md AC-01>
- **User Story**: <reference>
- **Precondition**: <setup>
- **Steps**:
  1. <step 1>
  2. <step 2>
- **Expected Result**: <result>
- **Actual Result**: ___
- **Status**: PASS / FAIL / BLOCKED
- **Notes**: ___

### Scenario 2: <title from acceptance.md AC-02>
...

## Edge Cases
- [ ] <edge case 1>: <steps> → Expected: <result>
- [ ] <edge case 2>: <steps> → Expected: <result>

## Non-Functional Checks
- [ ] Page load time is acceptable (< N seconds)
- [ ] Error messages are user-friendly (correct language)
- [ ] No console errors in browser
- [ ] Responsive on mobile (if applicable)
- [ ] Accessibility basics (keyboard navigation, screen reader)

## Bugs Found

| ID | Severity | Description | Steps to Reproduce | Status |
|----|----------|-------------|-------------------|--------|
| UAT-001 | P1/P2/P3 | <desc> | <steps> | Open / Fixed / Deferred |

## UAT Decision

- [ ] **PASS**: All scenarios pass, no P1/P2 bugs open → Proceed to production
- [ ] **CONDITIONAL PASS**: Minor issues (P3 only), acceptable for release with known issues
- [ ] **FAIL**: P1/P2 bugs found → Fix and re-test

## Sign-Off

| Role | Name | Decision | Date |
|------|------|----------|------|
| PM / BrSE | ___ | PASS / FAIL | ___ |
| Tech Lead | ___ | PASS / FAIL | ___ |
```

## Bug Categorization During UAT

| Severity | Description | Action |
|----------|-------------|--------|
| **P1** (Blocker) | Core functionality broken, cannot proceed | Fix immediately, re-test (light-spec or micro) |
| **P2** (Major) | Important function broken, workaround exists | Fix before release, re-test |
| **P3** (Minor) | Cosmetic, edge case, non-critical | Can release with known issue, fix in next sprint |

## Fix Loop Process

```
Bug found in UAT
    ↓
PM creates bug report (using table above)
    ↓
Dev creates fix (micro or light-spec -- fast track)
    ↓
    ├── Branch: fix/<change-id>-uat-<bug-id>
    ├── Quick peer review (1 reviewer)
    └── Merge to develop → cherry-pick to pre-release/*
    ↓
Re-deploy staging
    ↓
PM re-tests the specific scenario
    ↓
Update UAT checklist status
```

## UAT Best Practices

1. **Prepare test data in advance** -- don't waste UAT time on setup
2. **Test on staging, never on production** -- staging should mirror production
3. **Use real-world scenarios** -- test with realistic data and workflows
4. **Document everything** -- screenshots, screen recordings for complex bugs
5. **Time-box UAT** -- set a deadline (typically 2-3 days for feature-spec, 1 week for full-spec)
6. **Don't gold-plate** -- focus on acceptance criteria, not wish-list features
7. **Regression check** -- verify existing features still work, not just the new one
