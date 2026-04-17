# Test Plan: <change-id>

## Change Reference
- **Design**: [design.md](./design.md)
- **Acceptance**: [acceptance.md](./acceptance.md)
- **Contracts**: [contracts/](./contracts/)
- **Mode**: full-spec

## Test Strategy

| Level | Scope | Tools | Coverage Target | Responsibility |
|-------|-------|-------|----------------|----------------|
| Unit | Functions, classes, components | _pytest / jest_ | >= 80% | Each team |
| Integration | API endpoints, DB, services | _pytest + testclient_ | Key paths | Backend team |
| E2E | Critical user flows | _Playwright_ | Top 3-5 flows | QA / Frontend |
| Performance | Load, response time | _locust / k6_ | Defined SLAs | DevOps |
| Security | OWASP checks | _manual + tools_ | All endpoints | Security lead |

## Unit Test Cases

| ID | Component | Scenario | Expected Result | Team |
|----|-----------|----------|-----------------|------|
| UT-01 | _component_ | _scenario_ | _result_ | _team_ |

## Integration Test Cases

| ID | Flow | Scenario | Expected Result |
|----|------|----------|-----------------|
| IT-01 | _API flow_ | _scenario_ | _result_ |

## E2E Test Cases

| ID | User Flow | Steps | Expected Result |
|----|-----------|-------|-----------------|
| E2E-01 | _flow name_ | _step sequence_ | _result_ |

## Edge Cases

- _Edge case 1_: _description and how to test_
- _Edge case 2_: _description and how to test_

## Coverage Targets by Component

| Component | Target | Rationale |
|-----------|--------|-----------|
| Auth / Security | 90% | Critical path |
| Core business logic | 80% | Standard |
| Data access | 80% | Standard |
| UI components | 70% | Snapshot + behavior |
| Utilities | 70% | Low risk |

## Test Data Strategy

_How test data is created, managed, and cleaned up._

## Test Environment Requirements

_What infrastructure/services are needed for integration and E2E tests._
