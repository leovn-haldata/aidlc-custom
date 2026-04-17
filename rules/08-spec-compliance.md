---
description: >
  Specification compliance rules. Documentation-first workflow, contract stability,
  traceability chain, artifact lifecycle (3-tier storage), and breaking change protocol.
  Reference this when changing specs, API contracts, DB schemas, or managing artifact lifecycle.
---

# Specification Compliance

## Documentation-First Workflow

- When changing specifications or requirements, update documentation BEFORE modifying code.
- All code changes must be traceable to a spec artifact (proposal, design, or task).
- If a change affects API contracts, DB schema, or data formats, update the spec first and get approval.

## Contract Stability

- Do NOT unilaterally change API contracts (request/response schemas).
- Do NOT unilaterally change DB schemas without a migration plan.
- Do NOT unilaterally change data format definitions (input or output).
- Do NOT change data contracts between modules without updating shared documentation.
- Breaking changes require:
  1. Documentation in an ADR
  2. Approval via `/aidlc-confirm`
  3. Migration plan
  4. Rollback procedure

## Traceability Chain

Every artifact must maintain links to its context:

```
Intake Note  →  Proposal  →  Design  →  Plan  →  Tasks  →  Code  →  Tests
     ↑              ↑           ↑          ↑         ↑         ↑        ↑
  change-id    change-id   change-id  change-id  task-id  task-id  AC-id
```

- Commit messages should reference the change ID and task ID.
- PR descriptions should link to the spec package.
- Test names should reference the acceptance criterion they verify.

## Artifact Lifecycle

| Status | Meaning | Storage Tier |
|--------|---------|-------------|
| DRAFT | Being created, not yet reviewed | `changes/active/` |
| IN_REVIEW | Submitted for quality gate | `changes/active/` |
| CONFIRMED | Confirmed via `/aidlc-confirm`, ready for implementation | `changes/active/` |
| IN_PROGRESS | Implementation underway | `changes/active/` |
| COMPLETED | Implemented, tested, released, and shrunk | `changes/released/<YYYY-QN>/` |
| ARCHIVED | Long-term storage, summary only in Git | `changes/archived/<YYYY>/` |

## 3-Tier Storage Rules

See `docs/spec-lifecycle.md` for full details.

### What stays in Git

- Final spec artifacts (proposal, design, release note)
- README summary (entry point for each change)
- Links to PRs, issues, test results
- Architecture decisions (ADRs)
- Data model and API contracts (if changed)

### What should NOT be committed to Git

- Raw AI output (long, not reproducible)
- Debug logs and notes
- Multiple draft versions (Git history handles this)
- Intermediate screenshots (binary, use external storage)
- Detailed test reports in HTML format (use CI artifacts)

### Shrink Rule

When a change is released (`/aidlc-release`):
1. Generate `README.md` summary and `links.md`
2. Keep only essential knowledge files (what + why)
3. Remove process artifacts (how -- code is the truth)
4. Move to `changes/released/<YYYY-QN>/`
5. Released directory is **read-only**; post-release fixes create new changes

### Archive Rule

When released changes are > 2 quarters old:
1. Zip the full released directory
2. Push to external storage (S3/GCS/Drive)
3. Replace with `summary.md` containing link to archive
4. Move to `changes/archived/<YYYY>/`
