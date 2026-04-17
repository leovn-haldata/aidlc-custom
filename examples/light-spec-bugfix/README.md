# Example: light-spec -- Fix Login Timeout Bug

## Scenario

Users report that the login page times out after 10 seconds on slow connections. The API endpoint `/auth/login` has a hardcoded 10-second timeout that should be configurable. This is a complex bug fix (needs impact tracing, touches multiple files).

## Why light-spec (not micro)?

- Root cause needs investigation (not a trivial 1-line fix)
- Touches 2-3 files (config, service, test)
- Need to trace impact on other endpoints that may share the same timeout
- Need to document the change for future reference

## Flow Used

```
/aidlc-intake "Fix login timeout bug -- users on slow connections get timeout errors"
  → Recommended: light-spec (complex bug fix, 2-3 files, < 200 LOC)
  → User confirms

/aidlc-spec fix-login-timeout
  → Generated: proposal.md, design.md, tasks.md

/aidlc-confirm fix-login-timeout
  → AI pre-review: PASS (no issues)
  → Human: CONFIRM

/aidlc-build fix-login-timeout
  → TDD: Write test for configurable timeout → implement → pass

/aidlc-review fix-login-timeout
  → TDD: PASS (100% coverage on changed code)
  → Code Review: 0 issues
  → Security: 0 issues
  → Tech Lead decision: Accept → merge to develop
```

## Artifacts Produced (Active)

```
aidlc-docs/changes/active/fix-login-timeout/
├── intake-note.md
├── proposal.md
├── design.md
├── tasks.md
├── build-summary.md
├── review-report.md
└── release-note.md
```

## After Release (Shrunk → Released Tier)

```
aidlc-docs/changes/released/2026-Q2/fix-login-timeout/
├── README.md          # Summary entry point
├── proposal.md        # Final version
├── release-note.md
└── links.md           # PR, test results
```

## Branch: `fix/fix-login-timeout` → develop

## Time: ~45 minutes total
