# AIDLC Custom -- Command Catalog

## Lifecycle Commands (Tier 1: Always Install)

These commands form the main development flow and are always installed in every project. Every task enters through `/aidlc-intake` and flows through the pipeline, with steps added or skipped based on the spec mode.

```
                    ┌─────────────────────────────────────────────┐
                    │            ALL MODES (except micro)          │
                    │                                              │
                    │   /aidlc-intake   (classify & scope)         │
                    │         ↓                                    │
                    │   /aidlc-spec     (generate artifacts)       │
                    │         ↓                                    │
                    │   /aidlc-confirm  (pre-build gate)           │
                    │         ↓                                    │
                    │ ┌─────────────────────────────────────┐      │
                    │ │ feature-spec & full-spec ONLY       │      │
                    │ │                                     │      │
                    │ │   /aidlc-plan  (tech planning)      │      │
                    │ │         ↓                           │      │
                    │ │   /aidlc-tasks (task breakdown)     │      │
                    │ └─────────────────────────────────────┘      │
                    │         ↓                                    │
                    │   /aidlc-build    (TDD implement)            │
                    │         ↓                                    │
                    │   /aidlc-review   (PR review: TDD+Code+Sec)  │
                    │         ↓                                    │
                    │    ┌────────────┐                            │
                    │    │  Decision  │                            │
                    │    ├── Accept ──│→ /aidlc-release            │
                    │    ├── Fix ─────│→ Dev fixes → /aidlc-review │
                    │    ├── Postpone─│→ Back to PM/BrSE           │
                    │    └── Reject ──│→ Back to /aidlc-spec       │
                    │               │                              │
                    └───────────────│──────────────────────────────┘
                                    │
                    ┌──────────────────────────────────────────────┐
                    │         full-spec: UAT / Staging             │
                    │                                              │
                    │   Deploy staging → UAT test                  │
                    │         │                                    │
                    │    Pass ↓            Fail → Fix Bug Loop     │
                    │         │                  (light-spec/micro)│
                    │   /aidlc-release    (deploy production)      │
                    └──────────────────────────────────────────────┘
```

| Command | Modes | Description |
|---------|-------|-------------|
| `/aidlc-intake` | light-spec, feature-spec, full-spec | Receive requirement, classify complexity, recommend spec mode |
| `/aidlc-spec` | light-spec, feature-spec, full-spec | Generate spec artifacts (3/5/8+ files depending on mode) |
| `/aidlc-confirm` | light-spec, feature-spec, full-spec | Pre-build confirmation gate with scaled checklist; confirm / revise / reject |
| `/aidlc-plan` | feature-spec, full-spec | Technical planning: architecture, domain model, ADRs |
| `/aidlc-tasks` | feature-spec, full-spec | Task breakdown with dependency management |
| `/aidlc-build` | All (except micro) | TDD implementation: RED → GREEN → IMPROVE |
| `/aidlc-review` | All (except micro) | Post-build PR review: TDD Expert + Code Reviewer + Security (parallel) |
| `/aidlc-release` | All (except micro) | Archive spec package, generate release notes, deploy |

## Support Commands (Tier 2: Install Based on Project Needs)

> `/aidlc-fix` is Tier 1 (always installed). The rest are Tier 2 (install if relevant).

| Command | Description | When to Install |
|---------|-------------|----------------|
| `/aidlc-setup` | Initialize project directory structure and config files | First-time only |
| `/aidlc-deploy` | Package, deploy, and verify (Dev/Staging/Production). Includes IaC generation | Has staging/production |
| `/aidlc-monitor` | Set up monitoring, metrics, alerts, dashboards, and incident runbooks | Needs observability |
| `/aidlc-hotfix` | Emergency production fix (P0/P1) with fast-track gates and PIR | Has production |
| `/aidlc-transfer` | Project transfer/handover checklist and verification | Receiving/giving project |
| `/aidlc-modify` | Impact analysis for modifications to existing features | Changing existing features |
| `/aidlc-brownfield` | Analyze existing codebase, generate static/dynamic models | Has existing code |

## Flow by Spec Mode

### micro (2 steps -- no spec overhead)
```
fix → commit (optional: quick peer review)
```

### light-spec (6 steps)
```
intake → spec → confirm → build → review → release
```

### feature-spec (8 steps)
```
intake → spec → confirm → plan → tasks → build → review → release
```

### full-spec (9+ steps)
```
intake → spec → confirm → plan → (analyze) → tasks → build → review → release → deploy
```

## Role Mapping (Team Workflow)

| Role | Responsibilities | Commands |
|------|-----------------|----------|
| **JP PM / BrSE** | Define requirements, customer liaison, UAT validation | `/aidlc-intake`, `/aidlc-spec`, UAT |
| **Tech Lead** | Confirm specs, review PRs, architecture decisions, deploy oversight | `/aidlc-confirm`, `/aidlc-plan`, `/aidlc-tasks`, `/aidlc-review`, `/aidlc-deploy` (Staging+Prod), `/aidlc-release` |
| **Dev Team** | Implement features, write tests, fix bugs | `/aidlc-build`, `/aidlc-fix` |
| **AI Agent** | Generate specs, run reviews, produce reports | `/aidlc-spec`, `/aidlc-review` (3 experts) |
