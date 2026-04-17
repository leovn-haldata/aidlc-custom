# SKILLS Lifecycle Management

## Overview

SKILLS = the set of **rules**, **commands**, **templates**, and **AGENTS.md** that define how an AI agent operates in a project. This document defines how SKILLS evolve across projects and over time.

```
┌──────────────────────────────────────────────────────────┐
│                  Common SKILLS (shared)                   │
│                                                          │
│  aidlc-custom/  ← latest version, maintained by team     │
│  rules/ commands/ templates/ docs/ AGENTS.md             │
└──────────────┬───────────────────────────┬───────────────┘
               │ fork at project start     │ improvements merged back
               ▼                           ▲
┌──────────────────────┐     ┌──────────────────────┐
│  Project A SKILLS    │     │  Project B SKILLS    │
│  .cursor/rules/      │     │  .cursor/rules/      │
│  .cursor/commands/   │     │  .cursor/commands/   │
│  + project-specific  │     │  + project-specific  │
│    customizations    │     │    customizations    │
└──────────────────────┘     └──────────────────────┘
```

## Two Layers of SKILLS

### Common SKILLS (Organization-wide)

- Lives in: `aidlc-custom/` (this repository or a dedicated shared repo)
- Contains: Framework rules, commands, templates, docs, examples
- Maintained by: All teams via retrospective-driven proposals
- Versioned: Git tags (`skills-v1.0`, `skills-v1.1`, etc.)

### Project SKILLS (Per-project customization)

- Lives in: Each project's `.cursor/rules/`, `.cursor/commands/`, etc.
- Contains: Common SKILLS (as baseline) + project-specific additions
- Examples of project-specific additions:
  - Per-folder coding rules (`backend/rules/python.md`, `frontend/rules/react.md`)
  - Project-specific agent context in `AGENTS.md` (tech stack, domain knowledge)
  - Custom templates adapted to the project's conventions
  - Additional commands for project-specific workflows

## Lifecycle Flow

### 1. New Project Start

```
1. Run /aidlc-setup — installs only relevant files (Tier 1 + selected Tier 2)
   └── See README.md "Installation Tiers" for what gets installed

2. Customize for the project
   ├── Update AGENTS.md with project context (tech stack, domain)
   ├── Add per-folder coding rules (backend/rules/, frontend/rules/)
   ├── Adjust templates if project conventions differ
   └── Add project-specific commands if needed

3. Record the Common SKILLS version used
   └── Note in project README or AGENTS.md:
       "Based on aidlc-custom skills-v1.2"
```

### 2. During Development (Per Sprint)

```
Sprint in progress
    │
    ├── Team uses SKILLS daily (rules, commands, templates)
    │
    ├── Issues encountered?
    │   ├── Command produces poor output → note for retro
    │   ├── Rule too strict / too loose → note for retro
    │   ├── Missing template → create project-specific one
    │   └── Missing command → create project-specific one
    │
    └── Sprint Retrospective
        └── Review SKILLS effectiveness (see retro template)
```

### 3. Sprint Retrospective → SKILLS Proposals

At each retrospective, the team evaluates SKILLS effectiveness and generates improvement proposals:

```
Retrospective discussion
    │
    ├── Project-only improvement?
    │   └── Apply directly to project SKILLS (no approval needed)
    │
    └── Beneficial for all projects?
        └── Create a SKILLS Improvement Proposal (SIP)
            ├── Submit to Common SKILLS backlog
            ├── Review at cross-project sync meeting
            └── If approved → merge into Common SKILLS
```

### 4. Common SKILLS Update Cycle

```
Cross-project sync (monthly or quarterly)
    │
    ├── Review all submitted SIPs
    │   ├── Approve → merge into aidlc-custom/
    │   ├── Reject → document reason
    │   └── Defer → revisit next cycle
    │
    ├── Release new Common SKILLS version
    │   └── Tag: skills-vX.Y
    │
    └── Notify all projects of new version
        └── Each project decides when to sync
```

### 5. Syncing Common SKILLS to Projects

When a new Common SKILLS version is released:

```
New Common SKILLS version available (skills-v1.3)
    │
    ├── Compare with project's current SKILLS
    │   └── diff aidlc-custom/ vs project .cursor/
    │
    ├── Apply non-conflicting updates
    │   └── New rules, improved templates, new commands
    │
    ├── Review conflicting updates
    │   └── Project customization vs common change → team decides
    │
    └── Update version reference
        └── "Based on aidlc-custom skills-v1.3"
```

## SKILLS Improvement Proposal (SIP)

Use the appropriate format based on team size:

### SIP-Lite (teams < 5 people)

For small teams, a lightweight format in the sprint retrospective is sufficient:

```markdown
## SKILLS Proposals (in sprint-retrospective.md)

| What to Change | Why | Scope |
|----------------|-----|-------|
| Rule 04: add Go immutability example | Current examples are JS-only | Common |
| Add /aidlc-archive command | Manual archive is tedious | Project-only for now |
```

Decision: discuss at retro, Tech Lead approves, apply immediately. No formal review cycle needed.

### SIP-Full (teams >= 5 people or cross-project)

For larger teams or changes that affect Common SKILLS:

```markdown
# SIP: <title>

## Type
- [ ] New rule
- [ ] Rule modification
- [ ] New command
- [ ] Command improvement
- [ ] New template
- [ ] Template improvement
- [ ] New doc / guide
- [ ] Other: ___

## Problem
<What issue did you encounter? What was the impact?>

## Proposed Change
<What specifically should change in Common SKILLS?>

## Origin
- **Project**: <project name>
- **Sprint**: <sprint number>
- **Discovered by**: @name
- **Date**: YYYY-MM-DD

## Applicability
- [ ] All projects would benefit
- [ ] Most projects would benefit
- [ ] Niche -- only certain project types

## Example
<Show before/after or provide a concrete scenario>

## Decision
- **Status**: Proposed / Approved / Rejected / Deferred
- **Decided by**: <name>
- **Date**: YYYY-MM-DD
- **Reason**: <if rejected or deferred>
```

### When to Use Which

| Situation | Format |
|-----------|--------|
| Small team, project-only change | SIP-Lite (retro table row) |
| Small team, propose to common | SIP-Lite → discuss at cross-project sync |
| Large team or multi-project impact | SIP-Full |
| Breaking change to common SKILLS | SIP-Full (always) |

## Versioning Strategy

Use semantic versioning for Common SKILLS:

| Change Type | Version Bump | Example |
|-------------|-------------|---------|
| Typo, formatting fix | PATCH (1.0.0 → 1.0.1) | Fix typo in template |
| New doc, new template, non-breaking rule tweak | MINOR (1.0.0 → 1.1.0) | Add estimation guide |
| Breaking rule change, command behavior change | MAJOR (1.0.0 → 2.0.0) | Restructure spec modes |

```bash
git tag -a skills-v1.1.0 -m "Add SKILLS lifecycle, token tracking, commit-per-gate"
```

## Cursor Agent Skills (SKILL.md)

In addition to rules, commands, and templates, the framework includes **Cursor Agent Skills** — specialized `SKILL.md` files that give the AI agent deep expertise for specific workflows.

### Available Skills

| Skill | Location | Enhances |
|-------|----------|----------|
| `aidlc-tdd-build` | `skills/aidlc-tdd-build/SKILL.md` | TDD RED-GREEN-IMPROVE cycle during `/aidlc-build` |
| `aidlc-spec-review` | `skills/aidlc-spec-review/SKILL.md` | Spec validation + cross-spec conflict detection during `/aidlc-confirm` |
| `aidlc-shrink-archive` | `skills/aidlc-shrink-archive/SKILL.md` | Spec lifecycle management during `/aidlc-release` |

### How Skills Differ from Rules & Commands

| Component | Applied When | Purpose |
|-----------|-------------|---------|
| **Rules** (`.mdc`) | Automatically, on every interaction | Governance constraints, coding standards |
| **Commands** (`.md`) | User invokes explicitly (`/aidlc-*`) | Step-by-step workflow orchestration |
| **Skills** (`SKILL.md`) | Agent reads when task matches description | Deep domain expertise for specific tasks |

Skills are complementary: a **command** defines *what steps to follow*, a **skill** teaches the agent *how to do each step well*.

### Installation

Skills can be installed at two levels:

- **Project-level**: `.cursor/skills/aidlc-tdd-build/SKILL.md` — shared with the team
- **Personal**: `~/.cursor/skills/aidlc-tdd-build/SKILL.md` — only for your machine

The `/aidlc-setup` command offers to install skills during project initialization.

### Creating New Skills

Teams can create project-specific skills following the same pattern. Each skill needs:
1. A directory under `skills/` with a `SKILL.md` file
2. YAML frontmatter with `name` and `description`
3. Clear instructions (under 500 lines)

Improvements to skills follow the same SIP (SKILLS Improvement Proposal) process as other SKILLS components.

## Anti-Patterns

| Anti-Pattern | Why It's Bad | Correct Approach |
|-------------|-------------|-----------------|
| Never updating Common SKILLS | Teams reinvent the same fixes | Use retro → SIP → merge cycle |
| Force-syncing all projects immediately | Breaks project-specific customizations | Each project syncs on their schedule |
| Project never syncs with common | Misses improvements, diverges too far | Sync at least once per quarter |
| Dumping all project rules into common | Common becomes bloated and opinionated | Only promote truly universal improvements |
| No version tracking | Can't tell which common version a project uses | Always tag and record version |

## Integration with Existing Framework

| When | Action |
|------|--------|
| `/aidlc-setup` | Copy latest Common SKILLS as baseline |
| `/aidlc-transfer` (receive) | Check which SKILLS version the project uses; sync if outdated |
| Sprint Retrospective | Review SKILLS, generate SIPs (see retro template) |
| Cross-project sync | Review SIPs, update Common SKILLS, release new version |
| Maintenance mode | Continue using project SKILLS; sync common improvements quarterly |
