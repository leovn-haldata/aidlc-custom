---
description: >
  Testing requirements. TDD workflow, 80% minimum coverage, test types, and troubleshooting.
  Applied when working with test files.
globs:
  - "**/*.test.*"
  - "**/*.spec.*"
  - "**/tests/**"
  - "**/__tests__/**"
  - "**/test_*"
  - "**/*_test.*"
---

# Testing Requirements

## Minimum Coverage: 80%

- Overall: 80% minimum
- Critical paths (auth, data transformation, security): 90%+

## Test Types

1. **Unit Tests**: Individual functions, utilities, components
2. **Integration Tests**: API endpoints, database operations, external service interactions
3. **E2E Tests**: Critical user flows (Playwright recommended)

## TDD Workflow (Mandatory)

1. **RED**: Write test first -- it MUST fail
2. **GREEN**: Write minimal implementation to pass the test
3. **IMPROVE**: Refactor both code and test
4. **VERIFY**: Check coverage meets threshold

## Test Standards

- Mock external services in unit tests; use real services in integration tests
- Each test must be independent and repeatable
- Use descriptive test names: `test_<scenario>_<expected_outcome>`
- Test both success and error paths
- Verify authorization boundaries (authenticated vs unauthenticated, role-based)
- Cover edge cases: null, empty, boundary values, Unicode, concurrent access

## Troubleshooting Test Failures

1. Verify test isolation (no shared state between tests)
2. Verify mocks are correctly configured
3. Fix the implementation, not the test (unless the test is wrong)
4. If a test is flaky, find the root cause -- do not skip or retry blindly
