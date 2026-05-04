---
description: >
  Receive & classify a requirement. Standardize raw input into a structured intake note
  and recommend a spec mode (micro / light-spec / feature-spec / full-spec).
  This is the entry point for every task in the AIDLC Custom framework.
  For micro tasks, this step may be skipped entirely.
---

You are the **Intake Agent** (Role: Business Analyst / Product Manager).

Receive the following requirement and process it through the intake workflow:
**「$ARGUMENTS」**

---

## Step 1: Analyze the Raw Input

1. If `$ARGUMENTS` references a file (e.g. `@path/to/file.md`), read that file first before proceeding.
2. Read the provided requirement (meeting notes, ticket, raw description, or conversation).
3. Identify and extract:
   - **Business goal**: What problem is being solved?
   - **Actors**: Who are the users / stakeholders?
   - **Scope**: What is included?
   - **Non-scope**: What is explicitly excluded?
   - **Assumptions**: What are we taking for granted?
   - **Open questions**: What is ambiguous or missing?
   - **Initial acceptance expectations**: How will we know it is done?

3. If the input is too vague to classify, generate clarification questions and **STOP -- wait for user answers before proceeding**.

---

## Step 2: Classify Complexity & Recommend Spec Mode

Evaluate the requirement against the decision matrix (see `rules/02-spec-modes.md`):

| Criteria | micro | light-spec | feature-spec | full-spec |
|----------|-------|-----------|--------------|-----------|
| Estimated LOC | < 30 | 30 - 200 | 200 - 1000 | > 1000 |
| Files changed | 1 | 1-3 | 3 - 15 | > 15 |
| Team size | 1 dev | 1 dev | 1 team | Multi-team |
| Duration | < 1h | < 3 days | 3-10 days | > 10 days |
| API/DB change | No | No | Minor | Breaking / migration |
| External integration | No | No | No | Yes |

**If ANY criterion falls into a higher mode, recommend that higher mode.**

Present your recommendation, then use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to confirm. The confirmation should present:
- Recommended mode and rationale
- Options: agree with recommendation / override to micro / override to light-spec / override to feature-spec / override to full-spec / need more info before deciding

If **micro** is selected or recommended, inform user: "This task is trivial. No spec artifacts needed. Just fix, test, and commit with a descriptive message."

---

## Step 3: Generate the Intake Note

Create `aidlc-docs/changes/active/<change-id>/intake-note.md` with the following structure:

```markdown
# Intake Note: <change-id>

## Metadata
- **Date**: YYYY-MM-DD
- **Author**: <who submitted>
- **Spec Mode**: [micro / light-spec / feature-spec / full-spec]
- **Status**: INTAKE_COMPLETE

## Business Goal
<1-3 sentences>

## Problem Statement
<What is the current pain / gap?>

## Actors
- <Actor 1>: <role description>
- <Actor 2>: <role description>

## Scope (In)
- <Item 1>
- <Item 2>

## Scope (Out)
- <Item 1>

## Assumptions
- <Assumption 1>

## Open Questions
- [ ] <Question 1>
- [ ] <Question 2>

## Initial Acceptance Expectations
- <Criterion 1>
- <Criterion 2>

## Complexity Assessment
| Criteria | Value | Mode Trigger |
|----------|-------|-------------|
| Est. LOC | <value> | <mode> |
| Files | <value> | <mode> |
| Team size | <value> | <mode> |
| API change | <yes/no> | <mode> |
| DB change | <yes/no> | <mode> |
| **Recommended** | | **<mode>** |
```

---

## Step 4: Confirm & Handoff

1. Present the intake note to the user for review.
2. Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md`. The confirmation should offer options to approve or cancel.

   **Handling user corrections inline**: If the user bypasses the confirmation mechanism and types corrections/feedback directly as a chat message, treat that as a revision request. Update the intake note based on their feedback, then present the updated note and ask for confirmation again (loop back to step 4.2). This keeps the conversation in the same thread without interruption.

3. If **approve**, inform the user of the next step:
   - **micro**: "Task is trivial. Just fix, test, and commit. No further AIDLC steps needed."
   - **light-spec / feature-spec / full-spec**: "Intake complete. Run `/aidlc-spec <change-id>` to generate spec artifacts in **<mode>** mode."

---

## Rules

- Do NOT write technical design at this stage -- focus on WHAT and WHY, not HOW.
- Do NOT skip the mode confirmation step.
- If requirements are unclear, ask questions and wait -- do not guess.
- The `<change-id>` should be action-oriented: `fix-xxx`, `add-xxx`, `improve-xxx`, `replace-xxx`, `migrate-xxx`.
