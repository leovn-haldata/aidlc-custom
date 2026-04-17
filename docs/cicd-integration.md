# CI/CD Integration Guide

## Overview

This guide maps the AIDLC framework's quality gates to CI/CD pipeline checks, enabling automated enforcement alongside human review.

> **Related**: For per-change deployment execution (IaC generation, health checks, deploy reports), use `/aidlc-deploy`. This document covers **project-wide pipeline configuration**.

## Pipeline Architecture

```
Developer pushes code
        ↓
┌──────────────────────────────────────────────┐
│  CI Pipeline (automated)                      │
│                                               │
│  1. Lint check                                │
│  2. Type check (tsc / mypy / etc.)            │
│  3. Unit tests + coverage report              │
│  4. Security scan (secrets, dependencies)     │
│  5. Build verification                        │
│  6. Spec artifact validation (optional)       │
│                                               │
│  All pass? → PR ready for human review        │
│  Any fail? → Block merge, notify developer    │
└──────────────────────────────────────────────┘
        ↓
┌──────────────────────────────────────────────┐
│  Human Review (/aidlc-review equivalent)      │
│                                               │
│  Tech Lead reviews PR                         │
│  Decision: Accept / Request Fixes / Reject    │
└──────────────────────────────────────────────┘
        ↓
┌──────────────────────────────────────────────┐
│  CD Pipeline (automated after merge)          │
│                                               │
│  develop → Deploy to Dev Env                  │
│  pre-release/* → Deploy to Staging            │
│  master → Deploy to Production                │
└──────────────────────────────────────────────┘
```

## Gate Mapping: AIDLC → CI/CD

| AIDLC Gate | CI/CD Equivalent | Automated? |
|------------|-----------------|------------|
| `/aidlc-confirm` | PR template checklist + required labels | Partially (template auto-applied, human checks) |
| `/aidlc-review` (TDD Expert) | Coverage threshold check (≥ 80%) | Yes |
| `/aidlc-review` (Code Reviewer) | Lint + type check + PR review | Partially (lint auto, review human) |
| `/aidlc-review` (Security Reviewer) | Secret scan + dependency audit + SAST | Yes |
| `/aidlc-build` verification | Build + test pipeline | Yes |
| `/aidlc-deploy` | CD pipeline with environment gates | Yes (with manual approval for production) |

## PR Template (Auto-Applied)

Configure your repository to auto-apply this PR template:

```markdown
## Change Reference
- **Change ID**: CHG-xxx
- **Spec Mode**: micro / light-spec / feature-spec / full-spec
- **Spec Location**: `aidlc-docs/changes/active/<change-id>/`

## Checklist

### All modes
- [ ] Tests pass locally
- [ ] No hardcoded secrets
- [ ] Commit messages reference change ID

### light-spec and above
- [ ] Spec artifacts exist (`proposal.md`, `design.md`, `tasks.md`)
- [ ] `/aidlc-confirm` passed (confirmation report exists)
- [ ] Build summary generated (`build-summary.md`)

### feature-spec and above
- [ ] Test plan exists (`test-plan.md`)
- [ ] Acceptance criteria defined (`acceptance.md`)
- [ ] Architecture decisions documented

### full-spec only
- [ ] Data model reviewed (`data-model.md`)
- [ ] API contracts validated (`contracts/`)
- [ ] Rollout plan approved (`rollout-plan.md`)
```

## PR Labels (Auto-Labeling)

Auto-label PRs based on branch name or spec mode:

| Branch Pattern | Label | Color |
|---------------|-------|-------|
| `fix/*` | `micro` or `light-spec` | gray / blue |
| `feat/*` or `feature/*` | `feature-spec` | green |
| `hotfix/*` | `hotfix` | red |

## GitHub Actions Example

```yaml
name: AIDLC CI Pipeline

on:
  pull_request:
    branches: [develop, main, master]

jobs:
  lint-and-type:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: <install command>
      - name: Lint
        run: <lint command>
      - name: Type check
        run: <type check command>

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: <install command>
      - name: Run tests with coverage
        run: <test command with coverage>
      - name: Check coverage threshold
        run: |
          # Fail if coverage < 80%
          <coverage check command>

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Secret scan
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified
      - name: Dependency audit
        run: <audit command>

  build:
    runs-on: ubuntu-latest
    needs: [lint-and-type, test, security]
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: <build command>

  # Optional: validate spec artifacts exist for non-micro PRs
  spec-check:
    runs-on: ubuntu-latest
    if: "!startsWith(github.head_ref, 'fix/')"
    steps:
      - uses: actions/checkout@v4
      - name: Check spec artifacts
        run: |
          CHANGE_ID=$(echo "${{ github.head_ref }}" | sed 's|.*/||')
          SPEC_DIR="aidlc-docs/changes/active/${CHANGE_ID}"
          if [ ! -d "$SPEC_DIR" ]; then
            echo "WARNING: No spec directory found at $SPEC_DIR"
            echo "For light-spec and above, spec artifacts should exist."
          fi
```

## GitLab CI Example

```yaml
stages:
  - validate
  - test
  - security
  - build
  - deploy

lint:
  stage: validate
  script:
    - <lint command>

type-check:
  stage: validate
  script:
    - <type check command>

test:
  stage: test
  script:
    - <test command with coverage>
  coverage: '/TOTAL.*\s(\d+%)/'

security-scan:
  stage: security
  script:
    - <secret scan command>
    - <dependency audit command>

build:
  stage: build
  script:
    - <build command>

deploy-dev:
  stage: deploy
  script:
    - <deploy command>
  only:
    - develop
  environment: development

deploy-staging:
  stage: deploy
  script:
    - <deploy command>
  only:
    - /^pre-release\/.*/
  environment: staging
  when: manual

deploy-production:
  stage: deploy
  script:
    - <deploy command>
  only:
    - master
  environment: production
  when: manual
```

## Branch Protection Rules

### `develop`
- Require PR with at least 1 approval
- Require CI pipeline to pass (lint, test, security, build)
- No direct pushes (micro mode exempted by team policy)

### `pre-release/*`
- Require PR from develop only
- Require CI pipeline to pass
- Manual approval required for deployment

### `master` / `main`
- Require PR from pre-release/* or hotfix/* only
- Require 2 approvals (Tech Lead + 1 reviewer)
- Require all CI checks to pass
- No force pushes
- Exception: hotfix branches can merge directly with 1 approval

## Monitoring Integration

After deployment, CI/CD can trigger monitoring setup:

```yaml
post-deploy:
  script:
    - echo "Deployed $VERSION to $ENVIRONMENT"
    - curl -X POST $MONITORING_WEBHOOK -d '{"deployment": "$VERSION"}'
```

Track deployments in your monitoring tool to correlate deployments with metrics changes.
