---
description: >
  Core principles of the AIDLC Custom framework.
  AI-driven development lifecycle where AI decomposes workflows and generates recommendations;
  humans confirm and verify at defined gates. This rule is always applied.
alwaysApply: true
---

# AIDLC Custom -- Core Principles

2-layer AI-driven lifecycle: **Layer 1** = Governance & Quality Gates, **Layer 2** = Spec Execution (4 modes).

## Commands

| Command | Phase | Description |
|---------|-------|-------------|
| `/aidlc-intake` | Intake | Classify requirement, recommend spec mode |
| `/aidlc-spec` | Spec | Generate spec artifacts (mode-dependent) |
| `/aidlc-confirm` | Gate | Pre-build confirmation: confirm / revise / reject |
| `/aidlc-plan` | Plan | Technical planning (feature-spec & full-spec) |
| `/aidlc-tasks` | Tasks | Task breakdown (feature-spec & full-spec) |
| `/aidlc-build` | Build | TDD implementation |
| `/aidlc-review` | Review | Post-build PR review (TDD + Code + Security in parallel) |
| `/aidlc-release` | Release | Shrink spec, archive, generate release notes |
| `/aidlc-deploy` | Deploy | Package, deploy, verify (Dev/Staging/Production) |
| `/aidlc-monitor` | Ops | Monitoring, alerts, dashboards, runbooks |
| `/aidlc-hotfix` | Emergency | Production fix with fast-track gates (P0/P1) |
| `/aidlc-transfer` | Handover | Project transfer/handover checklist |
| `/aidlc-modify` | Support | Impact analysis for modifications |
| `/aidlc-fix` | Support | Build error resolution |
| `/aidlc-brownfield` | Support | Analyze existing codebase |
| `/aidlc-setup` | Setup | Initialize project structure |

## Spec Modes

| Mode | When | Duration |
|------|------|----------|
| **micro** | Typo, config, trivial 1-line fix | < 1 hour |
| **light-spec** | Complex bug fix, small feature (< 3 files) | < 3 days |
| **feature-spec** | Medium feature (1 team), enhancement | 3-10 days |
| **full-spec** | Large feature (multi-team), DB/API change | > 10 days |

See `rules/02-spec-modes.md` for the full decision matrix and artifact details.

## Fundamental Principles

1. **Wait for User**: At every gate or question, STOP and wait. Never proceed autonomously.
2. **Plan-First**: Create a plan (Markdown + checkboxes) and wait for human sign-off before executing.
3. **Quality Gates**: Spec confirmation (`/aidlc-confirm`), PR review (`/aidlc-review`), architecture decisions, deployment -- all require human sign-off.
4. **Context Memory**: All artifacts persist and feed subsequent steps. Never discard without explicit instruction.
5. **Traceability**: Every artifact links back: intake → spec → plan → task → code → test.
6. **Sprint Scope**: Respect committed sprint scope. No new features after Sprint Planning.
7. **Team Safety**: Respect spec locks, assigned tasks, and daily sync protocol. See `rules/03-team-workflow.md`.
8. **3-Tier Storage**: Active → Released (shrunk) → Archived. See `docs/spec-lifecycle.md`.
9. **Commit-per-Gate**: Commit after each major gate (spec, confirm, plan, build, review) so you can `git reset` to any gate boundary if issues are discovered later. Each commit message references the change ID and gate: `aidlc(CHG-xxx): spec complete` / `aidlc(CHG-xxx): confirm passed`.
