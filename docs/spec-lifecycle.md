# Spec Lifecycle & Storage Strategy

## Core Principle

> **Don't delete knowledge -- make it lighter and push it further away.**

- Git = lightweight source of truth
- External storage (S3/GCS/Drive) = full history and heavy artifacts

## 3-Tier Storage Model

```
aidlc-docs/changes/
├── active/                          # Tier 1: Currently implementing
│   ├── CHG-001-login-rate-limit/
│   ├── CHG-002-csv-export/
│   └── ...
├── released/                        # Tier 2: Completed, still needed for reference
│   ├── 2026-Q1/
│   │   ├── CHG-001-login-rate-limit/
│   │   └── ...
│   └── 2026-Q2/
│       └── ...
└── archived/                        # Tier 3: Long-term storage, rarely accessed
    ├── 2025/
    └── 2026/
        └── CHG-001-login-rate-limit/
            └── summary.md           # Single file + link to external storage
```

### Tier 1: Active (currently implementing)

- Contains all artifacts (spec, design, tasks, notes, debug, AI output...)
- Editable, continuously updated
- Scope: only changes actively worked on in the current sprint

### Tier 2: Released (completed, reference-ready)

- **Shrunk** version (see shrink process below)
- Read-only (no further edits)
- Organized by quarter: `released/2026-Q2/CHG-xxx/`
- Keep the most recent 2-4 quarters in Git

### Tier 3: Archived (long-term)

- Extremely lightweight: only 1 `summary.md` file in Git
- Full snapshot pushed to external storage (S3/GCS/Drive)
- Organized by year: `archived/2026/CHG-xxx/`

## Spec Shrink Process

When a change completes release, you **MUST** run shrink before moving to `released/`:

### Before shrink (active)
```
CHG-001-login-rate-limit/
├── intake-note.md
├── proposal.md
├── design.md
├── tasks.md
├── build-plan.md
├── build-summary.md
├── review-report.md
├── deploy-report.md
├── release-note.md
├── spec-plan.md
├── notes.md          ← noise
├── debug.md          ← noise
└── ai-drafts/        ← noise
```

### After shrink (released)
```
CHG-001-login-rate-limit/
├── README.md          # Standard summary (single entry point)
├── proposal.md        # Final, clean version
├── design.md          # Kept if contains architecture decisions
├── release-note.md    # Changelog entry
└── links.md           # Links to PR, test results, issues
```

### Shrink Rules

| Keep | Remove / move to external storage |
|------|----------------------------------|
| README.md (summary) | build-plan.md |
| proposal.md (final) | build-summary.md |
| design.md (if contains ADRs) | spec-plan.md |
| release-note.md | review-report.md |
| links.md (PR, test, issue) | deploy-report.md |
| data-model.md (if DB change) | notes.md, debug.md |
| contracts/ (if API change) | ai-drafts/, raw AI output |
| | tasks.md (completed tasks) |
| | intake-note.md |

**Principle**: Keep **what** and **why**. Remove **how** (code is the truth).

## README.md Template (Released)

Every change in `released/` MUST have a `README.md` -- this is the single entry point. 80% of lookups only need this file.

```markdown
# CHG-xxx: <Title>

## Summary
<1-3 sentences describing the change>

## Scope
- <bullet points of main changes>

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
- **Test Results**: <link>
- **Sprint**: Sprint N

## Timeline
- **Created**: YYYY-MM-DD
- **Released**: YYYY-MM-DD
- **Spec Mode**: light-spec / feature-spec / full-spec
```

## Archive Flow (Tier 2 → Tier 3)

When a change has been in `released/` long enough (typically > 2 quarters):

### Step 1: Create snapshot
```bash
cd aidlc-docs/changes/released/2026-Q1/CHG-001-login-rate-limit/
zip -r CHG-001-login-rate-limit.zip .
```

### Step 2: Push to external storage
```bash
gsutil cp CHG-001-login-rate-limit.zip gs://aidlc-archive/2026/Q1/
# or
aws s3 cp CHG-001-login-rate-limit.zip s3://aidlc-archive/2026/Q1/
```

### Step 3: Replace with summary + link
```
archived/2026/CHG-001-login-rate-limit/
└── summary.md
```

Contents of `summary.md`:
```markdown
# CHG-001: Login Rate Limit

## Summary
Added rate limiting to login endpoint to prevent brute-force attacks.

## Archive Location
- **Full archive**: gs://aidlc-archive/2026/Q1/CHG-001-login-rate-limit.zip
- **Archived on**: 2026-07-15

## Contains
- Full spec history (proposal, design, tasks)
- AI outputs and drafts
- Test logs and debug notes
- Review and deploy reports

## Quick Reference
- **PR**: #123
- **Sprint**: Sprint 3
- **Spec Mode**: light-spec
- **Released**: 2026-03-20
```

## What NOT to Commit to Git

This is the primary cause of repo bloat when using AI workflows:

| Don't commit | Reason | Alternative |
|-------------|--------|-------------|
| Long raw AI output | Noise, not reproducible | Keep only final artifact |
| Debug logs | Temporary, not needed | Delete after fix |
| Experimental prompts | Iteration noise | Keep final prompt in prompts.md |
| Multiple draft versions | Only need the final | Git history stores diffs |
| Intermediate screenshots | Binary, bloats repo | Upload to storage, keep link |
| Temp files (.tmp, .bak) | Not needed | `.gitignore` |
| Detailed test reports (HTML) | Heavy, generated | Upload to CI/storage |

## Sprint Backlog ≠ Spec Storage

Sprint backlog is a **lightweight index** pointing to changes:

```markdown
# Sprint 2026-S07

## Changes in this Sprint
- [ ] [CHG-015](../changes/active/CHG-015-user-profile/) @dev-a -- feature-spec
- [ ] [CHG-016](../changes/active/CHG-016-fix-timeout/) @dev-b -- light-spec
- [ ] [CHG-017](../changes/active/CHG-017-typo-header/) @dev-c -- micro

## Velocity: 21 pts (target: 24)
## Status: IN_PROGRESS
```

Sprint files do NOT contain full specs. When a sprint ends, the sprint file can be deleted or lightly archived. Knowledge lives in **changes**, not sprints.

## When External Storage Is Not Yet Needed

If your team:
- < 10 developers
- < 50 changes / month
- Not generating heavy AI output yet

Then 3-tier **within Git** is sufficient. Tier 3 keeps `summary.md` but no need to push to S3.

## Automation (When Scaling)

When needed, create scripts/commands:

1. **`/aidlc-archive <change-id>`** (future):
   - Zip entire change folder
   - Upload to S3/GCS
   - Generate `summary.md` with link
   - Remove excess files from Git
   - Update archive link

2. **Periodic cleanup** (monthly/quarterly):
   - Scan `released/` for changes > 2 quarters old
   - Propose archive candidates
   - Wait for approval
   - Execute archive flow
