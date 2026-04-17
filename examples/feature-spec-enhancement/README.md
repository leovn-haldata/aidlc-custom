# Example: feature-spec -- Add CSV Export Feature

## Scenario

Users need to export survey analysis results as CSV files. The feature requires a new API endpoint, background job processing, and a download button in the UI. Estimated: ~5 days, 1 team, ~8 files.

## Flow Used

```
/aidlc-intake "Add CSV export for survey results -- users need to download analysis data"
  → Recommended: feature-spec (new feature, 1 team, ~8 files, new API endpoint, 5 days)
  → User confirms

/aidlc-spec add-csv-export
  → Generated: proposal.md, design.md, tasks.md, test-plan.md, acceptance.md

/aidlc-confirm add-csv-export
  → AI pre-review: 1 WARNING (missing error handling for large files)
  → Human: CONFIRM with note to address warning

/aidlc-plan add-csv-export
  → Architecture: Background job with GCS storage, pre-signed URL for download
  → ADR: Choose background job over streaming (for files > 10MB)

/aidlc-tasks add-csv-export
  → 3 phases, 7 tasks, 2 parallel markers

/aidlc-build add-csv-export
  → TDD for each task: service → API → background job → UI button

/aidlc-review add-csv-export
  → TDD: PASS (85% coverage)
  → Code Review: 1 SUGGESTION (extract CSV formatter to utility)
  → Security: PASS (file download uses pre-signed URL, no direct path exposure)
  → Tech Lead decision: Accept → merge to develop
```

## Artifacts Produced

```
aidlc-docs/changes/active/add-csv-export/
├── intake-note.md
├── proposal.md
├── design.md
├── tasks.md
├── test-plan.md
├── acceptance.md
├── plan.md
├── build-summary.md
└── review-report.md
```

## Branch: `feature/add-csv-export` → develop (PR with 1 reviewer)

## Time: ~5 days total
