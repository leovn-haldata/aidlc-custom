# Multi-Repo Workflow Guide

This document describes the implementation details for using AIDLC across multiple separate repositories (e.g., backend / api-gateway / frontend).

## Overview

Multi-repo projects use an **orchestration hub** to coordinate specs and task breakdown, while each code repo maintains its own implementation and local docs.

```
Hub Repo (sdd-repo)               Code Repos
═══════════════════               ══════════
aidlc-docs/changes/               tv-backend/  .aidlc/link.md → hub
  active/<change-id>/             tv-api/      .aidlc/link.md → hub  
    intake-note.md                tv-app/      .aidlc/link.md → hub
    design.md (all repos)
    plan.md (repo boundaries)
    tasks.md ([REPO: xxx] tags)
    contracts/ (frozen contracts)
```

## Orchestration Hub Structure

All spec artifacts, plans, and task breakdowns live in the hub — not inside any individual code repo:

```
aidlc-docs-repo/
  changes/active/<change-id>/
    intake-note.md
    design.md          ← lists Repo Impact for all repos
    plan.md            ← lists Repo Boundaries + merge order  
    contracts/         ← frozen API/event/schema contracts
    tasks.md           ← tasks tagged [REPO: <name>]
    build-summary.md   ← lists all PRs, one per repo
    review-report.md
    release-note.md
```

## Branch Strategy (per repo)

For each affected repo, create the same branch name:

```
feature/<change-id>   ← created in backend, api-gateway, frontend
```

This makes it easy to cross-reference: searching `feature/add-csv-user-import` in any repo finds the related work.

## Merge Order

Always merge in dependency order — lower-layer repos first:

```
backend → api-gateway → frontend
```

Verify CI passes after each merge before merging the next repo. A failure in an intermediate repo must be resolved before continuing.

## Project Config Files

Two config files wire the multi-repo setup together:

### `.aidlc/config.md` (hub repo)

Lives in the hub repo. Declares all repos, their paths, and merge order. Created by `/aidlc-setup`.

```markdown
## Project
- Name: TV  
- Type: multi-repo
- Hub: /var/www/html/sdd-repo

## Repos
| Name       | Path (from hub) | Role                   | Merge Order |
|------------|----------------|------------------------|-------------|
| tv-backend | ../tv-backend  | Batch + business logic | 1           |
| tv-api     | ../tv-api      | REST API layer         | 2           |
| tv-app     | ../tv-app      | Frontend               | 3           |

## Merge Order
tv-backend → tv-api → tv-app
```

### `.aidlc/link.md` (code repos)

Lives inside each **code repo**. Points back to the hub. Created by `/aidlc-setup`.

```markdown
# AIDLC Link
- Hub: /var/www/html/sdd-repo
- Repo Name: tv-backend
- Role: Batch + business logic
```

## AI Tool Split + Handoff Protocol

```
[Claude Code — hub repo]          [Cursor — code repo]
        │                                  │
  /aidlc-intake                           │
  /aidlc-spec                             │
  /aidlc-plan                             │
  /aidlc-tasks                            │
        │                                  │
        │  tasks.md generated              │
        │  [REPO: tv-backend] tasks ───────┤ open tv-backend/ in Cursor
        │  [REPO: tv-api] tasks    ────────┤ open tv-api/ in Cursor
        │  [REPO: tv-app] tasks    ────────┤ open tv-app/ in Cursor
        │                                  │
        │                         /aidlc-build TV-001
        │                         reads .aidlc/link.md
        │                         → hub = /var/www/html/sdd-repo
        │                         → filter tasks [REPO: tv-backend]
        │                         → creates PR in tv-backend
        │                                  │
  /aidlc-review (reads all PRs)           │
  /aidlc-release (merges in order)        │
```

### Step-by-step handoff

1. **Governance phase** — Run from Claude Code inside the hub repo:
   - `/aidlc-intake`, `/aidlc-spec`, `/aidlc-plan`, `/aidlc-tasks`
   - Claude Code reads `.aidlc/config.md` to know all repos and paths
   - Generates `tasks.md` with tasks grouped and tagged by `[REPO: <name>]`

2. **Implementation phase** — For each repo group, switch to Cursor:
   - Open the code repo (e.g., `tv-backend/`) in Cursor
   - Run `/aidlc-build TV-001`
   - Build agent reads `.aidlc/link.md` → finds hub path → loads tasks → filters `[REPO: tv-backend]`
   - Implements only tv-backend tasks, creates branch + PR in tv-backend

3. **Repeat step 2** for each repo (tv-api, then tv-app)

4. **Review and release** — Switch back to Claude Code in hub:
   - Run `/aidlc-review TV-001` — reads all PRs listed in `build-summary.md`
   - Run `/aidlc-release TV-001` — merges in the order defined in `.aidlc/config.md`

## Contract Stability Across Repos

When tasks in different repos share an interface (API shape, event schema, shared type):

1. **Contract-first**: Define the contract in `plan.md` → `Repo Boundaries` section **before** coding starts
2. **Freeze contracts**: Use `/aidlc-plan` to create and freeze `contracts/` folder
3. **Parallel coding**: Both sides code against this agreed contract independently
4. **Ordered merge**: The lower-layer repo merges first; the upper-layer repo runs integration tests after
5. **Change control**: Any contract change mid-sprint triggers `/aidlc-modify` and re-confirmation from all affected devs

## Contract-First Development

See `/aidlc-plan` Step 4: Contract Freeze for the detailed contract freeze process.

Key principles:
- API/event/schema contracts must be **frozen before task breakdown**
- Contracts live in `contracts/` folder in the hub repo
- Each contract specifies provider, consumers, and exact schema
- Contract changes after freeze require updating all affected task packets and tests

## Team Coordination

- **Daily sync**: Check for cross-repo dependency changes
- **Merge coordination**: Follow the dependency order strictly  
- **Contract disputes**: Escalate to Tech Lead for resolution
- **Integration tests**: Run after each repo merge to validate contracts

For team workflow rules and conflict prevention, see `rules/03-team-workflow.md`.