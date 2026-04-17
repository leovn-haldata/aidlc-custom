---
description: >
  Spec mode decision matrix and artifact definitions. Defines 4 spec modes
  (micro / light-spec / feature-spec / full-spec), their artifacts, flows,
  review gates, branch strategies, and 3-tier storage convention.
  Reference this when classifying tasks, selecting modes, or checking artifact requirements.
---

# Spec Mode Selection Guide

## Decision Matrix

Use the following criteria to determine which spec mode to apply:

| Criteria | micro | light-spec | feature-spec | full-spec |
|----------|-------|-----------|--------------|-----------|
| Estimated LOC | < 30 | 30 - 200 | 200 - 1000 | > 1000 |
| Files changed | 1 | 1-3 | 3 - 15 | > 15 |
| Team size | 1 dev | 1 dev | 1 team | Multi-team |
| Duration | < 1 hour | < 3 days | 3-10 days (1-2 sprints) | > 10 days (> 2 sprints) |
| API contract change | No | No | Minor / additive | Breaking / new |
| DB schema change | No | No | Minor | Major / migration |
| External integration | No | No | No | Yes |
| Risk level | Trivial | Low | Medium | High |

**Selection rule**: If ANY criterion falls into a higher mode's column, use that higher mode.

**Override**: Users may force a higher mode than recommended, but should not go below the recommended minimum without explicit justification.

---

## Mode 0: micro (No Spec)

**Use for**: Typo fix, config change, dependency version bump, single-line bug fix, cosmetic CSS fix, comment fix. Tasks where the overhead of even a lightweight spec would exceed the value.

### Artifacts: None

No spec artifacts generated. The commit message serves as the only documentation.

### Flow (2 steps)

```
Fix → Commit (with descriptive message referencing ticket/issue if applicable)
```

Optional: If the change touches logic (not just cosmetic), run a quick peer review or self-review before pushing.

### Branch Strategy
- Branch: `fix/<short-description>` or commit directly on develop (team policy)
- Merge: Direct push or 1-click PR (no formal review gate)

### When NOT to use micro
- If the bug fix requires understanding impact on other modules → use light-spec
- If you're not 100% sure of the root cause → use light-spec
- If the change modifies behavior (not just appearance) → use light-spec

---

## Mode 1: light-spec

**Use for**: Complex bug fix (needs impact tracing), small feature (< 3 days), small refactor with behavior change, dependency upgrade with API change.

### Artifacts (3 files)

| File | Contents | Template |
|------|----------|----------|
| `proposal.md` | Business context, problem statement, scope, non-scope | OpenSpec `/opsx:new` style |
| `design.md` | Technical approach, impacted files, risks, backward compat | OpenSpec `/opsx:ff` design |
| `tasks.md` | Simple task checklist, no dependency graph needed | OpenSpec `/opsx:ff` tasks |

### Flow (6 steps)

```
/aidlc-intake → /aidlc-spec → /aidlc-confirm → /aidlc-build → /aidlc-review → /aidlc-release
```

### Review Gates
- **Pre-build** (`/aidlc-confirm`): Lightweight async -- AI pre-review + human confirm
  - Checklist: 5 items (justification, impact, risk, backward compat, test approach)
- **Post-build** (`/aidlc-review`): AI parallel review (TDD + Code + Security) + Tech Lead decision

### Branch Strategy
- Branch: `fix/<change-id>` or `feat/<change-id>`
- Merge: PR with Tech Lead approval, merge to develop

---

## Mode 2: feature-spec

**Use for**: Medium feature (3-10 days, single team), enhancement, new API endpoint (no breaking contract change), new UI feature.

### Artifacts (5 files)

| File | Contents | Template |
|------|----------|----------|
| `proposal.md` | Business context + detailed success criteria | Enhanced OpenSpec |
| `design.md` | Technical plan, architecture decisions, component interactions | Hybrid |
| `tasks.md` | Tasks with dependency graph, parallel markers `[P]`, file paths | Spec Kit style |
| `test-plan.md` | Test strategy, test case list, coverage target per component | AIDLC TDD Expert |
| `acceptance.md` | Acceptance criteria traceable to user stories | Spec Kit spec.md section |

### Flow (8 steps)

```
/aidlc-intake → /aidlc-spec → /aidlc-confirm → /aidlc-plan → /aidlc-tasks → /aidlc-build → /aidlc-review → /aidlc-release
```

### Review Gates
- **Pre-build** (`/aidlc-confirm`): Standard -- AI review + human meeting (15-30 min)
  - Checklist: 10 items (all light-spec items + architecture fit, test coverage plan, acceptance criteria, performance impact, security impact)
- **Post-build** (`/aidlc-review`): AI parallel review (TDD + Code + Security) + Tech Lead PR decision

### Branch Strategy
- Branch: `feature/<feature-id>`
- Merge: PR with at least 1 reviewer approval, merge to develop

---

## Mode 3: full-spec

**Use for**: Large feature (> 10 days, multi-team), major DB schema change, breaking API change, new system integration, migration, greenfield module, platform change.

### Artifacts (8+ files)

| File | Contents | Template |
|------|----------|----------|
| `proposal.md` | Full business case, stakeholder analysis, cost-benefit | Spec Kit style |
| `design.md` | System architecture, component design, sequence diagrams | Spec Kit plan.md |
| `tasks.md` | Full dependency graph, team assignment, sprint mapping | Spec Kit tasks.md |
| `test-plan.md` | Multi-level test strategy (unit, integration, E2E) | AIDLC + Spec Kit |
| `acceptance.md` | Full acceptance matrix, edge cases, rollback criteria | Spec Kit |
| `research.md` | Tech research, PoC results, trade-off analysis | Spec Kit research.md |
| `data-model.md` | DB schema, entity relationships, migration scripts | Spec Kit data-model.md |
| `contracts/` | OpenAPI spec, event contracts, interface contracts | Spec Kit contracts/ |
| `rollout-plan.md` | Phased rollout, feature flags, rollback plan, monitoring setup | AI-DLC Operations |

### Flow (9+ steps)

```
/aidlc-intake → /aidlc-spec → /aidlc-confirm → /aidlc-plan → (analyze) → /aidlc-tasks → /aidlc-build → /aidlc-review → /aidlc-release → /aidlc-deploy
```

### Review Gates
- **Pre-build** (`/aidlc-confirm`): Formal -- AI review + cross-team review meeting (30-60 min)
  - Checklist: 15+ items (all feature-spec items + data model review, contract review, rollout plan, rollback plan, monitoring plan, cross-team dependency sign-off)
- **Post-build** (`/aidlc-review`): AI parallel review (TDD + Code + Security) + Tech Lead PR decision
  - Additional: UAT / customer acceptance test (JP PM/BrSE + Tech Lead)

### Branch Strategy
- Branch: `feature/<feature-id>` + sub-branches `feature/<feature-id>/<task-id>`
- Merge: Squash merge after full review cycle
- Deploy flow: develop → pre-release/* (staging) → master (production)

---

## Decision Rule: When to Use Spec Kit (full-spec)?

If any of the following apply, use **full-spec**:

- [ ] Feature > 10 days of development
- [ ] Multiple teams involved
- [ ] Complex API / data contract
- [ ] High traceability required (audit, compliance)
- [ ] New greenfield module

**Principle**: Don't apply heavy framework overhead to every task. Use the right tool for the right job.

---

## Artifact Storage Convention (3-Tier)

All spec artifacts follow a 3-tier lifecycle. See `docs/spec-lifecycle.md` for full details.

### Tier 1: Active (currently implementing)
```
aidlc-docs/changes/active/<change-id>/
├── intake-note.md        (from /aidlc-intake)
├── proposal.md           (light / feature / full)
├── design.md             (light / feature / full)
├── tasks.md              (light / feature / full)
├── test-plan.md          (feature / full only)
├── acceptance.md         (feature / full only)
├── research.md           (full only)
├── data-model.md         (full only)
├── contracts/            (full only)
│   ├── api-spec.yaml
│   └── event-spec.md
├── rollout-plan.md       (full only)
├── plan.md               (from /aidlc-plan, feature / full)
├── build-plan.md         (from /aidlc-build)
├── build-summary.md      (from /aidlc-build)
├── review-report.md      (from /aidlc-review)
└── release-note.md       (from /aidlc-release)
```

### Tier 2: Released (completed, shrunk)
```
aidlc-docs/changes/released/<YYYY-QN>/<change-id>/
├── README.md             # Summary entry point (covers 80% of lookup needs)
├── proposal.md           # Final version
├── design.md             # Kept if contains ADRs
├── release-note.md       # Changelog
└── links.md              # PR, test results, issues
```

### Tier 3: Archived (long-term storage)
```
aidlc-docs/changes/archived/<YYYY>/<change-id>/
└── summary.md            # Single file + link to external storage
```

**Flow**: Active → (shrink) → Released → (archive + push S3/GCS) → Archived

---

## Quick Reference: Mode Selection Flowchart

```
Is it a typo, config tweak, or trivial 1-line fix?
  → YES → micro (no spec needed)

Is it a bug fix, hotfix, small feature < 3 days?
  → YES → light-spec

Does it involve multi-team coordination, DB migration, or breaking API changes?
  → YES → full-spec

Everything else (new feature 3-10 days, enhancement, new endpoint)
  → feature-spec
```

## Override Rules

- **Upward override**: Always allowed. If a dev feels a task needs more rigor, use a higher mode.
- **Downward override**: Requires justification. Document why the lower mode is sufficient.
- **Automatic escalation**: If during implementation you discover the task is more complex than initially assessed, escalate the mode and re-run `/aidlc-spec` with the higher mode.

## Artifact Comparison (All Modes)

| Artifact | micro | light-spec | feature-spec | full-spec |
|----------|-------|-----------|--------------|-----------|
| proposal.md | - | Brief | Detailed | Full business case |
| design.md | - | Impacted files | Architecture decisions | System architecture |
| tasks.md | - | Simple checklist | Dependency graph | Team-assigned with sprint mapping |
| test-plan.md | - | - | Test strategy | Multi-level strategy |
| acceptance.md | - | - | Given/When/Then | Full matrix with edge cases |
| research.md | - | - | - | Tech evaluation |
| data-model.md | - | - | - | Schema + migration |
| contracts/ | - | - | - | OpenAPI + events |
| rollout-plan.md | - | - | - | Phased rollout |
| **Total files** | **0** | **3** | **5** | **8+** |

## Common Scenario Examples

**micro**: Fix a typo in an error message, update a dependency version, change a config value, fix a CSS color, add a missing `alt` attribute.

**light-spec**: Fix a null pointer exception (need to trace impact), add a missing validation check, refactor with behavior change, fix a race condition, add a "copy to clipboard" button.

**feature-spec**: Add a "forgot password" flow, build a new dashboard page, add CSV export, implement a new search API endpoint, add caching to a service.

**full-spec**: Implement multi-tenant architecture, migrate SQL to NoSQL, add real-time collaboration, integrate with a payment system, major API version upgrade (v1 → v2).

## Quick Reference: Task Type → Mode Mapping

| Task Type | Mode | Reason |
|-----------|------|--------|
| Small bug fix | **micro** | Overhead not needed |
| Complex bug fix | **light-spec** | Need impact trace, lightweight proposal |
| Small feature (< 3 days) | **light-spec** | Change-centric, few artifacts |
| Medium feature (3-10 days) | **feature-spec** | Depends on complexity |
| Large feature (> 10 days) | **full-spec** | Full spec pipeline, contracts needed |
| New module / multi-team | **full-spec** | High traceability, clear API contracts |
| Large refactoring | **light-spec** or **feature-spec** | Change-centric, trace impact |
