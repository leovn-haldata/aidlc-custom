# Sprint Management Guide

## What is a Sprint?

A **sprint** is a time-boxed iteration (typically 1-2 weeks) with a committed scope, clear deliverables, and a defined deadline. Follows Scrum methodology.

```
Sprint 1: Foundation      → Sprint 2: Core Features → Sprint 3: Integration → Sprint 4: Polish
[DB + Auth + Base API]      [Business Logic + UI]     [External Services]     [Performance + UX]
```

For full-spec features spanning multiple sprints, use an **Epic** to group related sprints.

## Sprint Definition Template

```markdown
## Sprint N: <name>

### Scope
- **User Stories**: [US-01, US-02, US-03]
- **Changes**: [change-id-1, change-id-2]
- **Epic** (if applicable): <epic-name>

### Timeline
- **Start Date**: YYYY-MM-DD
- **End Date**: YYYY-MM-DD
- **Sprint Planning**: YYYY-MM-DD
- **Sprint Review / Demo**: YYYY-MM-DD
- **Retrospective**: YYYY-MM-DD

### Team
- **Scrum Master**: <name>
- **Product Owner**: JP PM / BrSE
- **Dev Team**: <names>

### Changes in this Sprint
- [ ] [CHG-015](../../changes/active/CHG-015-user-profile/) @dev-a -- feature-spec
- [ ] [CHG-016](../../changes/active/CHG-016-fix-timeout/) @dev-b -- light-spec
- [ ] [CHG-017](../../changes/active/CHG-017-typo-header/) @dev-c -- micro

### Sprint Backlog (tasks from changes above)
- [ ] Task 1 @dev-a (CHG-015)
- [ ] Task 2 @dev-b (CHG-016)
...

### Definition of Done
- [ ] All tasks completed
- [ ] All tests pass (coverage >= 80%)
- [ ] Code reviewed (no CRITICAL issues)
- [ ] Security reviewed
- [ ] Documentation updated
- [ ] Spec packages archived
- [ ] Demo-ready for Sprint Review

### Status
- [ ] NOT_STARTED
- [ ] IN_PROGRESS
- [ ] AT_RISK (with reason)
- [ ] COMPLETED
```

## Rules to Protect Sprint Scope

### Rule 1: Sprint Commitment

After Sprint Planning:
- NO new features added to the sprint backlog
- Bug fixes for existing scope: allowed (use micro or light-spec)
- Adjustments to existing tasks: allowed with Scrum Master approval
- New feature requests: go to the Product Backlog for next sprint

### Rule 2: Dependency Verification

Before starting a sprint:
1. List all dependencies from previous sprints
2. Verify each is COMPLETED (not just IN_PROGRESS)
3. If any dependency is not met: escalate to Product Owner
4. Never start implementation on unmet dependencies -- it always leads to rework

### Rule 3: Velocity-Based Planning

- Track team velocity (story points completed per sprint)
- Plan next sprint based on average velocity, not optimistic estimates
- Reserve ~15-20% capacity for:
  - Unexpected bugs found during implementation
  - Integration issues
  - Environment/tooling problems

### Rule 4: Early Escalation

| Signal | Action |
|--------|--------|
| 1 task behind schedule | Developer self-resolves, note in daily standup |
| 2+ tasks behind or critical task delayed | Escalate to Scrum Master |
| 30%+ capacity used before 50% completion | Escalate to Product Owner + Tech Lead |
| New dependency discovered mid-sprint | STOP, assess impact, may need re-plan |

Never wait until the sprint end to report a problem.

### Rule 5: No Cross-Sprint Leaking

- Do not work on tasks from next sprint during current sprint
- If sprint finishes early, use remaining capacity for:
  - Technical debt reduction
  - Test coverage improvement
  - Documentation updates
  - Refactoring (via light-spec or feature-spec)

## Sprint Health Tracking

```markdown
## Sprint Dashboard

| Sprint | Status | Progress | Velocity | Risk Level |
|--------|--------|----------|----------|------------|
| Sprint 1 | COMPLETED | 100% | 24 pts | - |
| Sprint 2 | IN_PROGRESS | 65% | - | LOW |
| Sprint 3 | NOT_STARTED | 0% | - | - |
```

Update at each daily standup.

## Scrum Ceremonies & AIDLC Mapping

| Scrum Ceremony | AIDLC Command | Timing |
|---------------|--------------|--------|
| Sprint Planning | `/aidlc-intake` + `/aidlc-tasks` + **conflict detection** | Sprint start |
| Daily Standup | Team sync checklist (rules/03) + **late conflict catch** | Daily |
| Sprint Review / Demo | `/aidlc-review` results + demo | Sprint end |
| Sprint Retrospective | N/A (team discussion) | After sprint review |
| Backlog Refinement | `/aidlc-intake` + `/aidlc-spec` | Mid-sprint |

### Sprint Planning: Conflict Detection Checkpoint

Sprint Planning doubles as the **primary checkpoint** for detecting cross-task conflicts:

1. **List all tasks** on a board/table.
2. **TL + team identify overlaps**: same module, API, DB table, or shared logic.
3. **For each overlap group** (10 min per group):
   - Agree on interface contract (function signatures, data shapes, error handling).
   - Document `[DEP: Task X.Y]` markers in `tasks.md`.
   - Decide merge order: lower layer first (DB → Service → API → UI).
4. **Independent tasks** → no constraints, proceed immediately after Planning.

**Principle**: Agree first → code parallel → merge ordered. Nobody waits for anyone else to finish coding.

Late-emerging conflicts are caught at **Daily Standup** (see `rules/03-team-workflow.md` § Daily Sync Checklist).

## Retrospective Questions (After Each Sprint)

1. Did we complete everything in the sprint backlog? If not, why?
2. What was our velocity? Is it trending up or down?
3. Were there any unexpected dependencies or blockers?
4. What would we do differently next sprint?
5. Should the next sprint's scope be adjusted based on velocity?

## Sprint Backlog as Lightweight Index

Sprint files are stored at `aidlc-docs/sprints/` and serve as **lightweight index files**, NOT as spec storage.

```
aidlc-docs/sprints/
├── sprint-2026-S06.md     # ← only recent sprints (2-4)
├── sprint-2026-S07.md
└── sprint-2026-S08.md     # ← current sprint
```

### Key Principles

- Sprint file = **index + links** to change directories
- Sprint file does NOT contain full specs (specs live in `changes/active/`)
- Old sprint files (> 4 sprints back) can be deleted or archived
- Knowledge lives in **changes**, not in sprints

### Sprint File Template

```markdown
# Sprint 2026-S08: <Sprint Goal>

## Status: IN_PROGRESS
## Duration: 2026-04-07 → 2026-04-18
## Velocity Target: 24 pts

## Changes
| Change | Owner | Mode | Status | Points |
|--------|-------|------|--------|--------|
| [CHG-020](../changes/active/CHG-020/) | @dev-a | feature-spec | IN_PROGRESS | 8 |
| [CHG-021](../changes/active/CHG-021/) | @dev-b | light-spec | DONE | 3 |
| [CHG-022](../changes/active/CHG-022/) | @dev-c | micro | DONE | 1 |

## Notes
- <standup notes, blockers, decisions>
```

### Why This Matters

| Approach | Repo size over time | Knowledge retention |
|----------|-------------------|-------------------|
| Full specs in sprint files | Grows fast, lots of duplication | Hard to find (scattered) |
| Sprint as index + changes/ | Stays small, no duplication | Easy to find (1 change = 1 folder) |
