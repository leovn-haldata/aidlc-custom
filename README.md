# AIDLC Custom Framework

A unified AI-driven development lifecycle framework combining **AI-DLC** (operating model) with **OpenSpec** and **Spec Kit** (execution layer) through 4 adaptive spec modes.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│           Layer 1: AI-DLC Operating Model           │
│  Governance · Quality Gates · Human-in-the-Loop     │
│  Lifecycle · Context Memory · Traceability          │
├─────────────────────────────────────────────────────┤
│           Layer 2: Spec Execution Layer             │
│  ┌────────┐ ┌──────────┐ ┌────────────┐ ┌───────┐  │
│  │ micro  │ │light-spec│ │feature-spec│ │full-  │  │
│  │ no-spec│ │ OpenSpec  │ │  Hybrid    │ │spec   │  │
│  │        │ │  style   │ │            │ │ SpecKit│  │
│  └────────┘ └──────────┘ └────────────┘ └───────┘  │
└─────────────────────────────────────────────────────┘
```

## 4 Spec Modes

| Mode | When to Use | Artifacts | Flow Steps | Time |
|------|-------------|-----------|------------|------|
| **micro** | Typo, config, trivial 1-line fix | 0 (commit msg only) | 2 | < 1h |
| **light-spec** | Complex bug fix, small feature < 3 days | 3 files | 6 | 1h - 3 days |
| **feature-spec** | Medium feature (3-10 days, 1 team), enhancement | 5 files | 8 | 3-10 days |
| **full-spec** | Large feature (> 10 days, multi-team), DB/API change | 8+ files | 9+ | 10+ days |

## Team Workflow (Aligned with Development Process)

```
                       ④ Spec & CHANGELOG (Single Source of Truth)
                       ════════════════════════════════════════════

┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐
│   JP PM / BrSE       │  │     Tech Lead         │  │     Dev Team         │
│   (Green Lane)       │  │     (Blue Lane)       │  │     (Yellow Lane)    │
├──────────────────────┤  ├──────────────────────┤  ├──────────────────────┤
│                      │  │                      │  │                      │
│ ① /aidlc-intake      │  │                      │  │                      │
│    Define Spec       │  │                      │  │                      │
│    (docs/*)          │  │                      │  │                      │
│         ↓            │  │                      │  │                      │
│ ② /aidlc-spec        │  │                      │  │                      │
│    Generate (AI)     │  │                      │  │                      │
│    (feat/*)          │  │                      │  │                      │
│         ↓            │  │                      │  │  Read & Confirm      │
│ ③ Customer Review ───│──│→ /aidlc-confirm      │  │         ↓            │
│    & Feedback        │  │  Validate & Confirm  │──│→ ⑤ /aidlc-build     │
│                      │  │                      │  │    Implement Feature │
│                      │  │                      │  │    (feature/*)       │
│                      │  │                      │  │         ↓            │
│                      │  │ ⑥ /aidlc-review     │←─│── Create PR         │
│                      │  │    Decision:         │  │                      │
│                      │  │    • Accept → Spec   │  │                      │
│                      │  │    • Fix → Dev ──────│──│→ (back to ⑤)        │
│                      │  │    • Postpone → PM   │  │                      │
│                      │  │    • Reject → Spec   │  │                      │
│                      │  │         ↓ Accept     │  │                      │
│                      │  │ ⑧ Merge → develop   │  │ ⑦ Merge → develop   │
│                      │  │    Deploy Dev Env    │  │    Deploy Dev Env    │
│                      │  │    (develop)         │  │    (develop)         │
│                      │  │         ↓            │  │                      │
│                      │  │ ⑨ /aidlc-deploy      │  │                      │
│                      │  │    Merge pre-release │  │                      │
│                      │  │    Deploy Staging    │  │                      │
│         ↓            │  │    (pre-release/*)   │  │                      │
│ ⑩ UAT / Test ────────│──│── (Shared)           │  │                      │
│    Validate spec &   │  │  Validate quality,   │  │                      │
│    customer needs    │  │  risk & readiness    │  │                      │
│         │            │  │                      │  │                      │
│    Pass ↓  Fail →────│──│──────────────────────│──│→ ⑪ Fix Bug Loop     │
│                      │  │                      │  │    (feature/*)       │
│                      │  │                      │  │    → Create PR fix   │
│                      │  │                      │  │    → Re-review (⑥)  │
│                      │  │ ⑫ Backup master      │  │                      │
│                      │  │         ↓            │  │                      │
│                      │  │ ⑬ /aidlc-release     │  │                      │
│                      │  │    Deploy Production │  │                      │
│                      │  │    (master)          │  │                      │
└──────────────────────┘  └──────────────────────┘  └──────────────────────┘

Note: For feature-spec & full-spec, /aidlc-plan (tech planning) and /aidlc-tasks
(task breakdown) run between Confirm and Build. Skipped for light-spec.
```

## Quick Start

### 1. Setup

Run `/aidlc-setup` — it asks about your project and installs only relevant files:

| Platform | Rules Location | Commands Location |
|----------|---------------|-------------------|
| Cursor | `.cursor/rules/` | `.cursor/commands/` |
| Google Antigravity | `.agents/rules/` | `.agents/workflows/` |
| Claude Code | `.claude/rules/` | `.claude/commands/` |

See **Installation Tiers** below for what gets installed vs. what stays as reference.

### 2. Workflow

Every task follows the same high-level flow, with steps added or skipped based on spec mode:

```
/aidlc-intake   →  Receive & classify requirement (all modes except micro)
      ↓
/aidlc-spec     →  Generate spec artifacts (mode-dependent)
      ↓
/aidlc-confirm  →  Pre-build gate: confirm / revise / reject (all modes except micro)
      ↓
/aidlc-plan     →  Technical planning (feature-spec & full-spec only)
      ↓
/aidlc-tasks    →  Task breakdown (feature-spec & full-spec only)
      ↓
/aidlc-build    →  TDD implementation (all modes except micro)
      ↓
/aidlc-review   →  PR Review: TDD + Code Review + Security (all modes except micro)
      ↓
/aidlc-release  →  Archive & release (all modes, depth varies)
```

### 3. Support Commands

| Command | Purpose |
|---------|---------|
| `/aidlc-setup` | Initialize project structure |
| `/aidlc-deploy` | Package, deploy, verify (Dev/Staging/Production) + IaC |
| `/aidlc-monitor` | Set up monitoring, alerts, dashboards, runbooks |
| `/aidlc-hotfix` | Emergency production fix with fast-track gates (P0/P1) |
| `/aidlc-transfer` | Project transfer/handover checklist & verification |
| `/aidlc-modify` | Impact analysis & modification planning |
| `/aidlc-fix` | Build error resolution |
| `/aidlc-brownfield` | Analyze existing codebase |

## Installation Tiers (75 files → ~25-38 in your project)

This repo contains **75 files**, but not all go into your project. They are organized into 3 tiers + optional skills:

### Tier 1: Core — Always Install (18 files)

Files that every AIDLC project needs. These form the governance + execution backbone.

| Category | Files | Count |
|----------|-------|-------|
| Rules (governance layer) | `rules/01-08*.md` | 8 |
| Lifecycle commands | `commands/aidlc-{intake,spec,confirm,plan,tasks,build,review,release}.md` | 8 |
| Essential support command | `commands/aidlc-fix.md` | 1 |
| Agent definitions | `AGENTS.md` (customize for your project) | 1 |

### Tier 2: Project Kit — Pick What You Need (10-35 files)

Copy based on your project's scale, spec modes, and operational needs.

| Category | Files | When to Copy |
|----------|-------|-------------|
| **Shared templates** | `templates/{intake-note,links,released-summary,archive-summary,gitignore-changes,project-config}.md` | Always (6 files) |
| **light-spec templates** | `templates/light-spec/{proposal,design,tasks}.md` | If using light-spec or higher (3 files) |
| **feature-spec templates** | `templates/feature-spec/{proposal,design,tasks,test-plan,acceptance}.md` | If using feature-spec or higher (5 files) |
| **full-spec templates** | `templates/full-spec/**` | If using full-spec (10 files) |
| **Sprint & UAT templates** | `templates/{sprint-retrospective,uat-checklist}.md` | If using sprints / UAT (2 files) |
| **CI template** | `templates/ci/{github-actions.yml,gitlab-ci.yml}` | Pick one for your CI platform (1 file) |
| **Support commands** | `commands/aidlc-{setup,deploy,monitor,hotfix,transfer,modify,brownfield}.md` | Copy what's relevant (0-7 files) |

### Cursor Agent Skills — Optional (3 skills)

Specialized `SKILL.md` files that enhance AI agent behavior during specific workflows.
Copy relevant skills to `.cursor/skills/` or `~/.cursor/skills/` for personal use.

| Skill | Purpose | Used By |
|-------|---------|---------|
| `aidlc-tdd-build` | TDD RED-GREEN-IMPROVE cycle | `/aidlc-build` |
| `aidlc-spec-review` | Spec completeness + cross-spec conflicts | `/aidlc-confirm` |
| `aidlc-shrink-archive` | Shrink to released tier, archive to S3/GCS | `/aidlc-release` |

### Tier 3: Reference Library — Read, Don't Copy (18 files)

These stay in the framework repo. Developers read them as needed.

| Category | Files | Purpose |
|----------|-------|---------|
| Guides & playbooks | `docs/*.md` (12 files) | How-to guides, processes, playbooks |
| Worked examples | `examples/*/README.md` (4 files) | Real-world mode examples |
| Framework docs | `README.md`, `commands/README.md` | Framework overview & catalog |

### Typical Project Sizes

| Project Type | Tier 1 | Tier 2 | Total |
|-------------|--------|--------|-------|
| Solo / light-spec only | 18 | ~10 (shared + light-spec) | **~28 files** |
| Small team / feature-spec | 18 | ~15 + 3 skills | **~36 files** |
| Multi-team / full-spec | 18 | ~27 + 3 skills | **~48 files** |

> **The `/aidlc-setup` command handles this automatically** — it asks about your spec modes and project needs, then copies only the relevant files.

## Directory Structure

```
aidlc-custom/                        # ← Framework source repo
│
│  ── Tier 1: Core ──────────────────
├── AGENTS.md                        # Agent & role definitions
├── rules/                           # Auto-applied governance rules (8 files)
├── commands/                        # Workflow commands (17 files)
│   ├── aidlc-{lifecycle}.md         # 8 lifecycle + 1 fix = 9 core
│   └── aidlc-{support}.md          # 7 support + README = 8 on-demand
│
│  ── Tier 2: Project Kit ───────────
├── templates/                       # Artifact templates (28 files)
│   ├── intake-note.md               # Shared templates (6 files)
│   ├── links.md
│   ├── released-summary.md
│   ├── archive-summary.md
│   ├── project-config.md
│   ├── gitignore-changes
│   ├── sprint-retrospective.md      # Sprint template
│   ├── uat-checklist.md             # UAT template
│   ├── light-spec/                  # 3 artifacts
│   ├── feature-spec/                # 5 artifacts
│   ├── full-spec/                   # 10 artifacts
│   └── ci/                          # CI pipeline templates
│       ├── github-actions.yml
│       └── gitlab-ci.yml
│
│  ── Cursor Agent Skills ──────────
├── skills/                          # Reusable AI agent skills (SKILL.md)
│   ├── aidlc-tdd-build/            # TDD cycle skill
│   ├── aidlc-spec-review/          # Spec review & conflict detection skill
│   └── aidlc-shrink-archive/       # Shrink & archive skill
│
│  ── Tier 3: Reference Library ─────
├── README.md                        # This file (framework overview)
├── docs/                            # 12 guides & playbooks
│   ├── adoption-guide.md
│   ├── developer-quickstart.md
│   ├── spec-lifecycle.md
│   ├── sprint-management.md
│   ├── team-conflict-playbook.md
│   ├── maintenance-guide.md
│   ├── cicd-integration.md
│   ├── uat-process.md
│   ├── versioning-guide.md
│   ├── estimation-guide.md
│   ├── skills-lifecycle.md
│   └── issue-tracker-integration.md
└── examples/                        # 4 worked examples
    ├── micro-trivial-fix/
    ├── light-spec-bugfix/
    ├── feature-spec-enhancement/
    └── full-spec-new-module/
```

## Spec Lifecycle (3-Tier Storage)

```
changes/active/     →  (shrink)  →  changes/released/    →  (archive)  →  changes/archived/
 Full artifacts          Keep what+why      Read-only, by quarter        Summary + S3/GCS link
```

- **Active**: Full working directory, editable
- **Released**: Shrunk to essentials (README, proposal, design, release note, links)
- **Archived**: 1 summary file in Git, full snapshot in external storage

Sprint backlog = lightweight index (links to changes, not full specs). See `docs/spec-lifecycle.md`.

## Key Principles

1. **Plan-First**: Every workflow creates a plan, waits for human sign-off
2. **Human-in-the-Loop**: Review gates at every mode, rigor scales with complexity
3. **Adaptive Complexity**: 4 modes -- use only the overhead the task demands (micro → full-spec)
4. **Context Memory**: All artifacts persist, serve as context for subsequent steps
5. **Traceability**: Every artifact links back to its requirement source
6. **Team Safety**: Spec locking, task assignment, sprint scope protection
7. **Knowledge Preservation**: Don't delete knowledge -- shrink it and push it further away
8. **Right Tool for Right Job**: Don't apply heavy framework overhead to every task

## Credits

Built on top of:
- [AI-DLC](https://github.com/awslabs/aidlc-workflows) by AWS Labs (Operating Model)
- [OpenSpec](https://github.com/Fission-AI/OpenSpec) by Fission-AI (Lightweight Spec Execution)
- [Spec Kit](https://github.com/github/spec-kit) by GitHub (Structured Spec Execution)
- [ai-dlc-sdd](https://github.com/tamagoya/ai-dlc-sdd) by tamagoya (Reference Implementation)
