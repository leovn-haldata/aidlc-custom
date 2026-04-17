# Team Conflict Playbook

## Prevention (Daily Practices)

### 1. Morning Sync (15 min max)

Each team member reports:
- What task am I working on today? (reference task ID)
- Am I blocked by anything?
- Am I touching files someone else is working on?

### 2. Spec Lock Discipline

Before editing ANY spec artifact:
1. Check for `<!-- LOCKED_BY: ... -->` in the file header
2. If unlocked, add your lock marker
3. Commit and push the lock immediately
4. Remove the lock when done editing
5. Locks stale for > 4 hours can be overridden (notify the holder first)

### 3. Task Ownership

- Only pick up `[UNASSIGNED]` tasks
- Mark `[IN_PROGRESS]` with your name immediately
- If you need to stop, mark `[BLOCKED]` with reason
- Never work on someone else's `[IN_PROGRESS]` task without asking

### 4. Frequent Integration

- Push changes at least once per day
- Pull before starting each session
- Keep branches short-lived (merge within 2-3 days for light-spec, 1 sprint for feature-spec)

---

## Resolution (When Conflicts Happen)

### Scenario 1: Two developers edited the same file

**Symptoms**: Git merge conflict on PR

**Resolution**:
1. Identify who has the "primary" change (based on task dependency)
2. Primary change merges first
3. Secondary developer rebases and resolves conflicts
4. If conflict is complex, pair-resolve with the other developer

**Prevention for next time**: Designate a "file lead" at morning sync when overlap is detected.

### Scenario 2: Spec conflict between two changes

**Symptoms**: Two change packages propose conflicting designs for the same module

**Resolution**:
1. Both specs go through `/aidlc-confirm` together
2. Tech Lead decides which takes priority
3. Lower-priority change adapts its design
4. Document the decision as an ADR

**Prevention**: Review the active spec list before creating new ones.

### Scenario 3: Sprint scope violation

**Symptoms**: A task implementation reaches into a module owned by a different sprint

**Resolution**:
1. STOP implementation immediately
2. Report to Scrum Master / Tech Lead
3. Options:
   a. Reorganize: Move the dependency into the current sprint (if small)
   b. Interface: Create a minimal interface; real implementation in the next sprint
   c. Replan: Adjust sprint scope via `/aidlc-modify`

**Prevention**: Explicit dependency check during Sprint Planning.

### Scenario 4: Blocked by another team

**Symptoms**: Task dependency on another team's deliverable that is delayed

**Resolution**:
1. Estimate the delay impact
2. If < 2 days: Use a stub/mock and continue; replace when dependency is ready
3. If > 2 days: Escalate to PM; consider re-ordering tasks or adjusting sprint scope
4. Document the block in the sprint impediments log

### Scenario 5: Technical disagreement

**Symptoms**: Two developers disagree on the right approach

**Resolution**:
1. Each writes a brief (3-5 sentences) rationale
2. Compare against existing ADRs and architecture patterns
3. If still unresolved: Tech Lead makes the call
4. Document the decision as an ADR
5. The "losing" approach is noted as a rejected option (not discarded)

### Scenario 6: Logic dependency conflict (different files, related logic)

**Symptoms**: Two devs work on separate tasks in the same sprint. They edit different files, so Git sees no conflict. But their changes are logically coupled -- one changes a data shape, validation rule, or interface that the other depends on. After both merge, runtime errors appear despite a clean Git merge.

**Example**: Dev A changes the response format of a service method. Dev B builds a new API endpoint that calls that same service method expecting the old format. Both PRs merge cleanly. Integration tests (if they exist) break. If they don't exist, production breaks.

**Resolution**:

1. **Identify the boundary**: Which shared interface, data contract, or function signature connects the two tasks?
2. **Agree on the contract first**: Both devs meet (10 min) and write down the shared interface:
   - Function signature(s)
   - Data shape (input/output types)
   - Validation rules
   - Error handling expectations
3. **Document as an Interface Contract** in the shared task or design.md:
   ```markdown
   ## Interface Contract: <boundary name>
   - **Between**: Task X.Y (@alice) ↔ Task X.Z (@bob)
   - **Interface**: `function_name(param: Type) -> ReturnType`
   - **Data shape**: { field: type, ... }
   - **Owner**: @alice (lower layer changes first)
   - **Consumer**: @bob (adapts after owner merges)
   ```
4. **Enforce merge order**: Lower-layer task merges first (DB → Service → API → UI). Upper-layer dev rebases after.
5. **Write a boundary integration test**: Test the specific interaction point between the two tasks. Assign this test to whichever dev finishes first, or create a shared task:
   ```markdown
   - [ ] Task X.9: Integration test for <boundary> @unassigned [UNASSIGNED]
     - Depends on: Task X.Y + Task X.Z
     - Validates: shared interface contract
   ```
6. **CI catch**: If integration tests exist, CI will catch the mismatch. If not, this scenario is exactly why you need them.

**Prevention (Sprint Planning + Incremental)**:

Sprint Planning is the **primary checkpoint** for detecting logic conflicts:

1. **At Sprint Planning** (mandatory):
   - TL + team review all tasks → identify which tasks touch shared modules/APIs/DB tables.
   - For each overlap group: 10 min sync → agree on interface contract → document `[DEP]` markers.
   - Decide merge order per group (lower layer first).
   - **After Planning**: each member proceeds independently. No one waits for others.

2. **During `/aidlc-tasks`** (per-change):
   - Mark tasks that share logic with `[DEP: Task X.Y]` marker (see task template).

3. **At daily sync** (safety net for late-emerging conflicts):
   - Add the question: **"Am I changing any interface or data shape that another task depends on?"**
   - If a NEW dependency is discovered: 10 min sync between affected devs → agree contract → continue parallel.

4. **Principle**: **Agree first → code parallel → merge ordered.**
   - Nobody waits for anyone else to finish coding.
   - Only merge has an order (lower layer first, so CI catches mismatches early).

**Anti-patterns**:
- **Batch-reviewing all specs before anyone codes** -- creates bottleneck, wastes time
- Assuming "different files = no conflict" -- logic conflicts are invisible to Git
- Both devs coding against an assumed (undocumented) interface
- Skipping integration tests because "we'll test manually"
- Merging both PRs on the same day without running integration tests between them

---

## Escalation Ladder

| Level | When | Who Decides | Timeframe |
|-------|------|------------|-----------|
| 1. Developer discussion | Same-file conflict, minor disagreement, logic dependency | Developers involved | Same day |
| 2. Tech Lead arbitration | Spec conflict, architecture disagreement, interface contract dispute | Tech Lead | Within 24h |
| 3. PM/Stakeholder input | Scope conflict, priority dispute, budget impact | PM + Tech Lead | Within 48h |
| 4. Architecture review | System-wide impact, breaking change | Architecture team | Scheduled meeting |
