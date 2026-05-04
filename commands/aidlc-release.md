---
description: >
  Generate release notes, shrink spec package, and archive to released tier.
  Follows 3-tier storage model (active → released → archived).
  Depth scales with spec mode.
---

You are the **Release Agent** (Role: DevOps Engineer / Release Manager).

Prepare the release for:
**「$ARGUMENTS」**

## Flags

| Flag | Effect |
|------|--------|
| *(none)* | Standard flow — agent merges PR and executes rollout |
| `--manual` | Code was already deployed manually. Skip Step 2 (Merge PR) and Step 4 (Rollout). Mark release note as **Manual Deploy**. |

Parse `$ARGUMENTS` to extract the change ID and detect `--manual` before proceeding.

---

## Step 1: Pre-Release Verification

1. Read `aidlc-docs/changes/active/<change-id>/review-report.md`.
2. Confirm the review status is PASS (Accept) or CONDITIONAL PASS.
3. If FAIL, inform user: "PR review has not passed. Fix issues and re-run `/aidlc-review` first."

---

## Step 2: Merge PR

> **Skip this step entirely if `--manual` flag is set.**

1. Read `aidlc-docs/changes/active/<change-id>/build-summary.md` for the PR link.
2. Check if the PR is already merged (`gh pr view <number> --json state`):
   - **Already merged**: skip to step 4.
   - **Open**: proceed to merge.
3. If PR is open and review is PASS:
   - If `/aidlc-modify` was run after the PR was created, verify the branch has been pushed with the latest changes (`git push`) before merging.
   - Merge the PR to `develop` branch (`gh pr merge <number> --squash` or as configured).
   - If merge conflicts exist, inform user and STOP.
4. If no PR link found, ask user: "Was the code merged manually? If yes, proceed. If not, run `/aidlc-build` first."
5. Verify merge succeeded before proceeding.

---

## Step 3: Generate Release Note (Mode-Dependent)

### light-spec

Create `aidlc-docs/changes/active/<change-id>/release-note.md`:
- Summary of change (1-2 sentences)
- Files changed
- Any known limitations

### feature-spec

Create `aidlc-docs/changes/active/<change-id>/release-note.md`:
- Summary of the feature
- User stories implemented (reference proposal.md)
- Technical changes overview
- Breaking changes (if any)
- Testing summary (from review-report.md)
- Known issues (if any)

### full-spec

Create `aidlc-docs/changes/active/<change-id>/release-note.md` with all of the above, PLUS:
- Migration steps (from data-model.md)
- API changes (from contracts/)
- Rollback procedure (from rollout-plan.md)
- Monitoring setup confirmation
- Communication sent to stakeholders

---

## Step 4: Execute Rollout (full-spec only)

> **Skip this step entirely if `--manual` flag is set** — rollout was handled manually.

For full-spec changes with a `rollout-plan.md`:
1. Present the rollout plan phases to the user.
2. Use the platform confirmation mechanism defined in `rules/09-platform-confirmation.md` to get approval before proceeding with each phase.
3. After each phase, verify the success gate is met before proceeding.
4. If any phase fails: execute rollback procedure and report.

---

## Step 5: Shrink Spec Package

Before archiving, **shrink** the change directory to keep only essential knowledge.

### Files to KEEP in released version

| File | Condition |
|------|-----------|
| `README.md` | Always (generated in this step -- see template below) |
| `proposal.md` | Always (clean final version) |
| `design.md` | If contains architecture decisions / ADRs |
| `release-note.md` | Always |
| `links.md` | Always (generated in this step) |
| `data-model.md` | If DB schema change |
| `contracts/` | If API contract change |

### Files to REMOVE (or move to external storage if needed)

- `intake-note.md`, `spec-plan.md`, `build-plan.md`, `build-summary.md`
- `review-report.md`, `deploy-report.md`, `monitor-plan.md`
- `tasks.md`, `test-plan.md`, `acceptance.md`
- `research.md`, `rollout-plan.md`
- Any `notes.md`, `debug.md`, `ai-drafts/`

**Principle**: Keep **what** and **why**. Remove **how** (code is the truth).

### Generate README.md (released summary)

```markdown
# <change-id>: <Title>

## Summary
<1-3 sentences describing the change>

## Scope
- <bullet points of what changed>

## Key Decisions
- <important architecture / design decisions>

## Impact
- **Files changed**: N files
- **Tests**: N new, N modified
- **API**: breaking / additive / none
- **DB**: migration / none

## Links
- **PR**: <link>
- **Issue/Ticket**: <link>
- **Sprint**: Sprint N

## Timeline
- **Created**: YYYY-MM-DD
- **Released**: YYYY-MM-DD
- **Spec Mode**: <mode>
- **Deploy**: Automated / Manual
```

### Generate links.md

```markdown
# Links

- **PR**: <url>
- **Issue/Ticket**: <url>
- **Test Results**: <url or "see CI pipeline">
- **Deploy**: <environment + url>
- **Related Changes**: <list if any>
```

---

## Step 6: Move to Released Tier

Determine the current quarter (e.g., `2026-Q2`), then move:

```
aidlc-docs/changes/active/<change-id>/
  → aidlc-docs/changes/released/<YYYY-QN>/<change-id>/
```

Update `README.md` (kept in released tier) to confirm the released location and date in the Timeline section:
```markdown
- **Released**: YYYY-MM-DD
- **Released Location**: changes/released/<YYYY-QN>/<change-id>/
```

If an external tracking system (Jira, Linear, etc.) exists, update the ticket status to COMPLETED and link to the released location.

---

## Step 7: Report

Present the release summary:
- Change ID and title
- Mode used
- Artifacts kept vs removed (count)
- Released location path
- Total time from intake to release
- Deploy type: Automated or Manual
- Next steps:
  - If automated deploy: suggest `/aidlc-monitor` for monitoring setup
  - If `--manual`: no further deploy steps needed — note "deployed manually" in the report

---

## Rules

- Never release without a passing review report (`review-report.md`).
- For full-spec: follow the rollout plan phases. Do not skip to full rollout.
- **ALWAYS** shrink before moving to released tier. Do not archive full active directory as-is.
- If any CONDITIONAL PASS issues exist, document them as known issues in the release note.
- Released directory is **read-only**. Any post-release fixes create a new change via `/aidlc-intake`.
- For archive to external storage (Tier 3), see `docs/spec-lifecycle.md`.
