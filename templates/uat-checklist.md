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
  1. <step>
- **Expected Result**: <result>
- **Actual Result**: ___
- **Status**: PASS / FAIL / BLOCKED
- **Notes**: ___

## Edge Cases
- [ ] <edge case 1>: Expected: <result>

## Non-Functional Checks
- [ ] Page load time acceptable
- [ ] Error messages are user-friendly
- [ ] No console errors in browser
- [ ] Responsive on mobile (if applicable)

## Bugs Found

| ID | Severity | Description | Steps to Reproduce | Status |
|----|----------|-------------|-------------------|--------|
| UAT-001 | P1/P2/P3 | <desc> | <steps> | Open / Fixed / Deferred |

## UAT Decision

- [ ] **PASS**: All scenarios pass → Proceed to production
- [ ] **CONDITIONAL PASS**: Minor P3 issues only → Release with known issues
- [ ] **FAIL**: P1/P2 bugs found → Fix and re-test

## Sign-Off

| Role | Name | Decision | Date |
|------|------|----------|------|
| PM / BrSE | ___ | PASS / FAIL | ___ |
| Tech Lead | ___ | PASS / FAIL | ___ |
