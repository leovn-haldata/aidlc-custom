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

### /aidlc-intake — Accepted Input Formats

No preparation required. Pass any of the following:

| Format | Example |
|--------|---------|
| Plain description | `/aidlc-intake add a CSV upload screen for bulk user registration` |
| Pasted content | `/aidlc-intake` + paste meeting notes, email, or Jira ticket |
| File reference | `/aidlc-intake @requirements/user-import.md` |
| Conversation summary | `/aidlc-intake the client wants a revenue dashboard filterable by region and date, exportable to PDF` |
| Mixed | `/aidlc-intake @meeting-notes/2026-05-04.md — focus on the user management section` |

> If input is too vague, the agent stops and asks clarifying questions. More context upfront = fewer questions.

---

## Support Commands (Tier 2: Install Based on Project Needs)

> `/aidlc-fix` is Tier 1 (always installed). The rest are Tier 2 (install if relevant).

| Command | Description | When to Install |
|---------|-------------|----------------|
| `/aidlc-setup` | Initialize project directory structure and config files | First-time only |
| `/aidlc-deploy` | Package, deploy, and verify (Dev/Staging/Production). Includes IaC generation | Has staging/production |
| `/aidlc-monitor` | Set up monitoring, metrics, alerts, dashboards, and incident runbooks | Needs observability |
| `/aidlc-hotfix` | Emergency production fix (P0/P1) with fast-track gates and PIR | Has production |
| `/aidlc-transfer` | Project transfer/handover checklist and verification | Receiving/giving project |
| `/aidlc-modify` | Impact analysis and update plan for changes to an **in-progress** change (same change ID) | Changing existing features |
| `/aidlc-brownfield` | Analyze existing codebase, generate static/dynamic models | Has existing code |

## aidlc-modify vs aidlc-intake

Use `/aidlc-modify` when a requirement **within an existing change** shifts mid-flight. Use `/aidlc-intake` when the request is a **separate, independent requirement**.

| | `/aidlc-modify` | `/aidlc-intake` (new task) |
|---|---|---|
| **Trigger** | Requirement changes mid-sprint, post-release bug, new constraint on existing feature | Brand new requirement |
| **Target** | Existing `active/<change-id>/` folder | Creates a new change ID |
| **Output** | `modification-plan.md` linked to original change | New spec folder from scratch |
| **Traceability** | Stays within the same change ID | Independent change |

**Rule of thumb**: if the change belongs to the same original requirement → `/aidlc-modify`. If it stands alone, even if related → `/aidlc-intake`.

> Example: mid-build, the client adds a filter to the export feature → `/aidlc-modify` (same change ID, updates design.md + tasks.md + acceptance.md).
> Example: the client requests a new reporting screen → `/aidlc-intake` (new change ID, even if it uses the same export logic).

---

## End-to-End Example (feature-spec)

Using a single feature — "CSV upload for bulk user registration" — as a running example:

```bash
# 1. Intake: describe the requirement in any form
/aidlc-intake add a screen for admins to upload a CSV file for bulk user registration

# Intake agent recommends: feature-spec
# Change ID assigned: add-csv-user-import

# 2. Generate spec artifacts (proposal, design, tasks, test-plan, acceptance)
/aidlc-spec add-csv-user-import

# 3. Dev runs AI pre-review → presents to TL for approval
/aidlc-confirm add-csv-user-import

# 4. Dev drafts architecture plan → TL approves
/aidlc-plan add-csv-user-import

# 5. Generate task breakdown with dependency graph
/aidlc-tasks add-csv-user-import

# 6. TDD implementation → self-check → create PR
/aidlc-build add-csv-user-import

# --- mid-sprint: client requests email validation before import ---
# Scope change within same feature → modify, not a new intake
/aidlc-modify add-csv-user-import client now requires email format validation before importing each row

# 7. TL runs official 3-expert review (TDD + Code + Security)
/aidlc-review add-csv-user-import

# 8. Merge PR, generate release note, archive spec package
/aidlc-release add-csv-user-import

# -- or, if code was already deployed manually (merge PR + pull code on server) --
/aidlc-release add-csv-user-import --manual
```

**Support commands** (when relevant):

```bash
# Production incident during rollout
/aidlc-hotfix INC-2026-042 -- CSV import endpoint returning 500 for files over 2MB

# Deploy to staging environment
/aidlc-deploy add-csv-user-import staging

# Set up monitoring after deploy
/aidlc-monitor add-csv-user-import

# Analyze existing codebase before starting (first time only)
/aidlc-brownfield

# Fix build errors after a merge
/aidlc-fix add-csv-user-import
```

---

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

**Principle**: Developer drives the flow and runs most commands — AI generates artifacts. Tech Lead approves at gate points and runs the official review.

| Role | Runs commands | Gate responsibility |
|------|--------------|-------------------|
| **PM / BrSE** | `/aidlc-intake` | UAT sign-off (full-spec) |
| **Developer** | `/aidlc-spec` `/aidlc-confirm` `/aidlc-plan` `/aidlc-tasks` `/aidlc-build` `/aidlc-release` | — |
| **Tech Lead** | `/aidlc-review` `/aidlc-hotfix` `/aidlc-deploy` (Prod) | Confirm gate · Plan output · Review per-issue decisions |
| **DevOps** | `/aidlc-deploy` `/aidlc-monitor` | — |

### Who does what per spec mode

| Step | micro | light-spec | feature-spec / full-spec |
|------|-------|-----------|--------------------------|
| intake | Dev | Dev | Dev (or PM for customer-facing requirements) |
| spec | — | Dev | Dev |
| confirm | — | Dev self-confirms | Dev runs → **TL approves** |
| plan | — | — | Dev runs → **TL approves** |
| tasks | — | — | Dev |
| build | Dev | Dev | Dev |
| review | — | **TL runs** (abbreviated) | **TL runs** (full 3-expert review) |
| release | Dev | Dev | Dev runs → **TL approves** |

> **light-spec**: TL review is abbreviated — if 0 CRITICAL + 0 WARNING findings, no per-issue decisions needed.
> **feature-spec and above**: TL runs full review with per-issue decisions (Fix / Dismiss / Defer).
> Dev's "self-check" is Step 6 of `/aidlc-build` (lint, tests, build, security scan) — not a separate `/aidlc-review` run.
