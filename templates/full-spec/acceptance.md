# Acceptance Criteria: <change-id>

<!-- Extends feature-spec acceptance format with:
     Edge Case Acceptance table, Rollback Criteria, E2E verification, and expanded Definition of Done -->

## Change Reference
- **Proposal**: [proposal.md](./proposal.md)
- **Mode**: full-spec

## Criteria Matrix

### AC-01: _Title linked to US-01_

**User Story**: US-01 -- As a _[actor]_, I want _[capability]_, so that _[benefit]_.

**Given** _precondition_
**When** _action_
**Then** _expected result_

**And** _additional condition_
**Then** _additional result_

- [ ] Verified in unit test: _test ID_
- [ ] Verified in integration test: _test ID_
- [ ] Verified in E2E test: _test ID_ (if applicable)

### AC-02: _Title linked to US-02_

**Given** _precondition_
**When** _action_
**Then** _expected result_

- [ ] Verified in unit test: _test ID_
- [ ] Verified in integration test: _test ID_

## Edge Case Acceptance

| ID | Scenario | Expected Behavior | Test ID |
|----|----------|-------------------|---------|
| EC-01 | _edge case_ | _behavior_ | _test_ |

## Rollback Criteria

_Under what conditions should the release be rolled back?_
- _Criterion 1_
- _Criterion 2_

## Definition of Done

- [ ] All acceptance criteria verified
- [ ] Unit tests pass (all teams)
- [ ] Integration tests pass
- [ ] E2E tests pass
- [ ] Code reviewed (no CRITICAL issues)
- [ ] Security reviewed (no CRITICAL issues)
- [ ] Performance validated against SLAs
- [ ] Documentation updated
- [ ] API contracts validated
- [ ] Data migration tested (up and down)
- [ ] Rollout plan reviewed
- [ ] Monitoring configured
- [ ] Build passes on all environments
