# Test Plan: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Acceptance**: [acceptance.md](./acceptance.md)
- **Mode**: feature-spec

## Test Strategy

| Level | Scope | Tools | Coverage Target |
|-------|-------|-------|----------------|
| Unit | Individual functions, classes | _pytest / jest / etc._ | >= 80% |
| Integration | API endpoints, DB operations | _pytest + testclient / supertest_ | Key paths |
| E2E | Critical user flows | _Playwright / Cypress_ | Top 2-3 flows |

## Key Test Cases

### Unit Tests

| ID | Component | Scenario | Expected Result |
|----|-----------|----------|-----------------|
| UT-01 | _component_ | _scenario_ | _result_ |
| UT-02 | _component_ | _scenario_ | _result_ |

### Integration Tests

| ID | Flow | Scenario | Expected Result |
|----|------|----------|-----------------|
| IT-01 | _API endpoint_ | _scenario_ | _result_ |

## Edge Cases to Cover

- _Edge case 1_: _description_
- _Edge case 2_: _description_

## Coverage Targets by Component

| Component | Target | Rationale |
|-----------|--------|-----------|
| _Auth_ | 90% | Security-critical |
| _Business logic_ | 80% | Standard |
| _Utils_ | 70% | Low risk |
