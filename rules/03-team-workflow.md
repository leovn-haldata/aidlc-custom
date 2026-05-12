---
description: >
  Team collaboration and conflict prevention rules for multi-developer projects.
  Defines spec locking, task assignment, branch strategy per mode, file conflict
  resolution, sprint scope protection, and daily sync checklist.
  Reference this when working in a team, resolving conflicts, or managing sprints.
---

# Team Workflow & Conflict Prevention

## 1. Spec Locking Protocol

When editing any spec artifact, add a lock marker to the file header:

```markdown
<!-- LOCKED_BY: dev-name | SINCE: YYYY-MM-DDTHH:MM -->
```

Rules:
- Before editing a spec file, check for an existing lock marker.
- If locked by someone else, do NOT edit. Coordinate with the lock holder.
- Remove the lock marker when your edit is complete and committed.
- Locks older than 4 hours without activity are considered stale and can be overridden after notifying the holder.

## 2. Task Assignment Protocol

In `tasks.md`, each task tracks ownership and status:

```markdown
- [x] Task 1: Setup database schema @alice [DONE]
- [ ] Task 2: Implement user service @bob [IN_PROGRESS]
- [ ] Task 3: Add API endpoints [UNASSIGNED]
- [ ] Task 4: Write integration tests @alice [BLOCKED] (waiting on Task 2)
```

Rules:
- Only pick up `[UNASSIGNED]` tasks.
- Mark as `[IN_PROGRESS]` with your name before starting.
- Mark as `[DONE]` when complete and tests pass.
- Mark as `[BLOCKED]` with reason if you cannot proceed.
- Do NOT reassign another dev's `[IN_PROGRESS]` task without their acknowledgment.

## 3. Branch Strategy by Mode

| Mode | Branch Pattern | Merge Target | Merge Strategy |
|------|---------------|-------------|----------------|
| micro | `fix/<short-desc>` or direct on develop | develop | Direct push or 1-click PR |
| light-spec | `fix/<change-id>` or `feat/<change-id>` | develop | PR with Tech Lead approval |
| feature-spec | `feature/<change-id>` | develop | PR with 1+ reviewer approval |
| full-spec | `feature/<change-id>` + `feature/<change-id>/<task-id>` | develop → pre-release/* → master | Squash merge after full review cycle |

### Deploy Flow (full-spec)
```
feature/* → develop (Dev Env) → pre-release/* (Staging) → master (Production)
```

Rules:
- Branch name must include the `<change-id>` for traceability.
- For full-spec sub-branches, merge into the feature branch first, then feature branch into develop.
- Never push directly to `main`/`master` or `develop` without a PR (micro mode exempted by team policy).
- `pre-release/*` branches are created for staging deployment and UAT testing.
- `master` receives merges only after UAT passes.

## 4. Conflict Prevention (Sprint Planning + Incremental)

### Primary checkpoint: Sprint Planning

Sprint Planning is the **first and most important** checkpoint for detecting conflicts:

1. **List all tasks** for the sprint on a board/table.
2. **TL + team identify overlaps**: which tasks touch the same module, API, DB table, or shared logic?
3. **For overlapping tasks** (10 min per group):
   - Agree on interface contract (function signatures, data shapes, error handling).
   - Document in `tasks.md` with `[DEP: Task X.Y]` markers.
   - Decide merge order: lower layer first (DB → Service → API → UI).
4. **Independent tasks** → no constraints, each member proceeds immediately.

**Principle**: Agree first → code parallel → merge ordered. Nobody waits for anyone else to finish coding.

### File Conflicts (same file, different tasks)

When the same file appears in multiple tasks:

1. **Detect at Sprint Planning** or daily sync.
2. **Designate a "file lead"** for that file.
3. **File lead** merges others' changes and resolves conflicts.
4. **Commit frequency**: Commit and push frequently (at least daily) to minimize merge conflicts.

### Logic Conflicts (different files, shared interface/data)

When two tasks modify different files but share a logical boundary:

1. **Detect at Sprint Planning**: TL reviews task list for shared modules/APIs.
2. **Agree on contract** at Planning (10 min sync per group) -- write down:
   - Function signature(s), data shape, validation rules, error handling
   - Owner (lower layer) and consumer (upper layer)
3. **Code in parallel**: Both devs work independently, following the agreed contract.
4. **Merge order**: Lower-layer merges first → CI validates → upper-layer rebases.
5. **Boundary test**: Integration test for the shared interface.

See `docs/team-conflict-playbook.md` Scenario 6 for full resolution process.

### Late-emerging conflicts (Daily Sync safety net)

Not all conflicts are visible at Sprint Planning. New ones surface during implementation:

- **Daily sync catch**: "Am I changing any interface or data shape that another task depends on?"
- **If discovered**: 10 min sync between affected devs → agree on contract → continue parallel.
- **Escalate** if > 2 tasks share the same boundary.

### Anti-patterns to avoid
- Waiting for all specs to be ready before anyone starts coding (bottleneck).
- Two devs editing the same function concurrently.
- Large, infrequent commits that create massive merge conflicts.
- Refactoring a shared file while another dev adds features to it.
- Assuming "different files = no conflict" -- logic conflicts are invisible to Git.
- Coding against an undocumented/assumed interface without agreeing first.

## 5. Sprint Scope Protection

A sprint has a committed scope after Sprint Planning:

```markdown
## Sprint N: <name>
- **Backlog Items**: [US-01, US-02, change-id-1]
- **Dependencies**: [Sprint N-1 items X, Y must be COMPLETED]
- **Start Date**: YYYY-MM-DD
- **End Date**: YYYY-MM-DD
- **Velocity Target**: N story points
- **Status**: NOT_STARTED | IN_PROGRESS | AT_RISK | COMPLETED
```

Rules:
- **Sprint Commitment**: After Sprint Planning, no new features are added. Bug fixes and adjustments only.
- **Dependency Gate**: Before starting a sprint, verify all dependency items are COMPLETED.
- **Capacity Reserve**: Reserve ~15-20% capacity for unexpected issues, not for scope creep.
- **Escalation**: If a sprint moves to AT_RISK status, escalate to the Scrum Master/Tech Lead immediately. Do not wait for sprint end.
- **No Cross-Sprint Work**: Do not implement features belonging to a future sprint in the current one.

## 6. Daily Sync Checklist

For team projects, verify daily:

- [ ] All `[IN_PROGRESS]` tasks have been worked on or status updated
- [ ] No stale spec locks (> 4 hours)
- [ ] No unresolved file conflicts
- [ ] No unresolved logic dependencies (check `[DEP: ...]` markers in tasks.md)
- [ ] **Am I changing any interface or data shape that another task depends on?** (late conflict detection)
- [ ] Current sprint status is accurate
- [ ] Any blockers are communicated
- [ ] Any NEW cross-task dependency discovered? → 10 min sync to agree contract

## 7. Conflict Resolution Escalation

| Situation | Action |
|-----------|--------|
| Two tasks modify the same file | Designate file lead at daily sync |
| Two tasks share logic boundary (different files) | Agree on interface contract, enforce merge order (see playbook Scenario 6) |
| Spec conflicts with another spec | Escalate to Tech Lead via `/aidlc-confirm` |
| Sprint scope violated | STOP work, re-plan via `/aidlc-modify` |
| Disagreement on technical approach | Create an ADR, discuss, Tech Lead decides |
| Blocked by another team | Escalate to PM, add to sprint impediments |

---

## 8. Multi-Repo Rules

For projects with multiple separate repositories:

### Orchestration Principle
- Use a dedicated hub repo for all spec artifacts and task coordination
- Code repos contain only code and local docs, not cross-repo specs
- All repos link back to the hub via `.aidlc/link.md`

### Branch Consistency
- Use identical branch names across all affected repos: `feature/<change-id>`
- Makes cross-referencing and tracking easier

### Merge Order Enforcement  
- **Always merge lower-layer repos first**: backend → api → frontend
- Verify CI passes after each repo merge before continuing
- Any merge failure blocks the entire sequence until resolved

### Contract Freeze Rule
- Define all cross-repo contracts **before** task breakdown begins
- Freeze contracts using `/aidlc-plan` Step 4 process
- Any contract change after freeze requires re-confirmation from all affected teams
- See `docs/multi-repo-workflow.md` for implementation details
