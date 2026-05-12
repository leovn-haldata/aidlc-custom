---
description: >
  Generate spec artifacts based on the selected spec mode (light-spec / feature-spec / full-spec).
  Reads the intake note and produces mode-appropriate deliverables using templates.
---

You are the **Spec Agent** (Role: Specification Writer).

Generate spec artifacts for the following change:
**「$ARGUMENTS」**

---

## Step 1: Load Context

1. Read the intake note at `aidlc-docs/changes/active/<change-id>/intake-note.md`.
2. Confirm the spec mode from the intake note's metadata.
3. If no intake note exists, inform the user: "Run `/aidlc-intake` first."
4. Read the relevant template files from `templates/<mode>/`.

---

## Step 2: Create Plan

Create a spec generation plan at `aidlc-docs/changes/active/<change-id>/spec-plan.md`:
- List each artifact to be generated with checkboxes
- Note any dependencies or open questions
- Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` before generating artifacts.

---

## Step 3: Generate Artifacts (Mode-Dependent)

### light-spec (3 artifacts)

**proposal.md** -- OpenSpec style:
- Business context (from intake note)
- Problem statement
- Scope / non-scope
- Success criteria (brief)

**design.md** -- OpenSpec style:
- Technical approach (concise)
- Repo Impact (list affected repos; omit if single-repo project):
  - `<repo-name>`: impacted files with paths
- Risks (1-3 items)
- Backward compatibility notes

**tasks.md** -- Simple checklist:
- Ordered task list
- Each task: description, target file(s), estimated effort
- No dependency graph needed

### feature-spec (5 artifacts)

All of light-spec, PLUS:

**proposal.md** -- Enhanced:
- Add detailed success criteria with measurable outcomes
- Add user story references

**design.md** -- Enhanced:
- Add architecture decisions (inline, no separate ADR)
- Add component interaction description
- Add data flow if applicable
- Repo Impact (required if cross-repo):
  - `<repo-name>`: impacted files with paths, PR branch name

**tasks.md** -- Spec Kit style:
- Dependency graph between tasks
- Parallel markers `[P]` for tasks that can run concurrently
- File paths for each task
- TDD structure: test tasks before implementation tasks

**test-plan.md** -- New:
- Test strategy (unit, integration, E2E scope)
- Key test cases with expected behavior
- Coverage targets per component
- Edge cases to cover

**acceptance.md** -- New:
- Acceptance criteria mapped to user stories
- Given/When/Then format for each criterion
- Definition of Done checklist

### full-spec (8+ artifacts)

All of feature-spec, PLUS:

**proposal.md** -- Full business case:
- Stakeholder analysis
- Cost-benefit summary
- Timeline expectations

**design.md** -- System architecture:
- Component diagram description
- Sequence flow for key scenarios
- Pattern choices with rationale

**research.md** -- New:
- Technology options evaluated
- PoC results (if any)
- Trade-off matrix
- Dependency version compatibility

**data-model.md** -- New:
- Entity relationship descriptions
- Table/collection schemas
- Migration plan (up and down)
- Data validation rules

**contracts/** -- New directory:
- `api-spec.yaml`: OpenAPI 3.x specification
- `event-spec.md`: Event contracts (if applicable)

**rollout-plan.md** -- New:
- Phased rollout strategy
- Feature flag configuration
- Rollback procedure
- Monitoring checkpoints
- Communication plan

---

## Step 4: Cross-Reference Check (feature-spec & full-spec)

After generating all artifacts:
1. Verify every acceptance criterion in `acceptance.md` has a corresponding task in `tasks.md`.
2. Verify every API endpoint in `contracts/` has a corresponding test case in `test-plan.md`.
3. Verify `design.md` references are consistent with `data-model.md` (full-spec).
4. List any gaps found and ask the user how to resolve them.

---

## Step 5: Handoff

1. Update the spec plan checkboxes.
2. Present a summary of all generated artifacts.
3. Inform the user of the next step:
   - "Spec complete. Run `/aidlc-confirm <change-id>` to start the pre-build confirmation gate."

---

## Rules

- Use templates from `templates/<mode>/` as structural guides, not rigid forms -- adapt to the specific change.
- Every artifact MUST reference the `<change-id>` and link back to `intake-note.md`.
- Do NOT add technical implementation details in `proposal.md` -- keep it business-focused.
- For full-spec, if research reveals the approach is infeasible, STOP and report to user before continuing.
- Check the `rules/08-spec-compliance.md` for contract stability rules before generating contracts.
