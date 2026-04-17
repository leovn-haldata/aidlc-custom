---
name: aidlc-shrink-archive
description: >-
  Shrink completed spec packages to released tier and archive old specs to external storage.
  Use when running /aidlc-release, cleaning up completed changes, or when the user asks
  to shrink, archive, or clean up spec artifacts.
---

# AIDLC Shrink & Archive

## When to Use

- User runs `/aidlc-release` (shrink happens as part of release)
- User asks to clean up completed specs
- Quarterly maintenance: archive released specs older than 2 quarters

## Shrink: Active → Released

### What to KEEP

| File | Why |
|------|-----|
| `README.md` | Summary entry point (generate if missing) |
| `proposal.md` | Business context (what + why) |
| `design.md` | Architecture decisions (keep if contains ADRs) |
| `release-note.md` | Changelog entry |
| `links.md` | Links to PR, test results, issues |
| `data-model.md` | Keep if DB schema changed |

### What to REMOVE

| File | Why safe to remove |
|------|-------------------|
| `intake-note.md` | Absorbed into proposal |
| `tasks.md` | Tasks done, code is the truth |
| `build-plan.md` | Process artifact, no longer needed |
| `build-summary.md` | Process artifact |
| `review-report.md` | Decision recorded in PR |
| `deploy-report.md` | Process artifact |
| `spec-plan.md` | Process artifact |
| `test-plan.md` | Tests in code are the truth |
| `acceptance.md` | Verified, results in PR |
| `notes.md`, `debug.md` | Noise |
| `ai-drafts/` | Not reproducible |

### Shrink Steps

```
1. Verify change is COMPLETED (all tasks done, review passed, merged)
2. Generate README.md if missing:
   - Change ID, title, date, mode, owner
   - One-paragraph summary
   - Link to PR, link to proposal
3. Generate links.md if missing:
   - PR URL, CI results, issue tracker link
4. Delete files not in KEEP list
5. Move directory:
   changes/active/CHG-xxx/ → changes/released/YYYY-QN/CHG-xxx/
6. Commit: aidlc(CHG-xxx): shrink to released tier
```

## Archive: Released → Archived

### When to Archive

- Released specs **older than 2 quarters**
- Example: In Q3 2026, archive everything in `released/2025-Q4/` and older

### Archive Steps

```
1. For each change in the target quarter:
   a. Zip the full released directory
   b. Upload to external storage (S3/GCS/Drive)
   c. Create summary.md:

      # CHG-xxx: <title>
      - **Date**: YYYY-MM-DD
      - **Mode**: <spec mode>
      - **Owner**: @name
      - **Result**: <one sentence>
      - **Archive**: <S3/GCS URL to zip>

   d. Replace directory with summary.md only:
      changes/archived/YYYY/CHG-xxx/summary.md

2. Commit: aidlc: archive YYYY-QN specs to external storage
```

## Verification

After shrink or archive, verify:

- [ ] No broken links in sprint files or CHANGELOG
- [ ] Released directory is read-only (no further edits expected)
- [ ] Archive links are accessible
- [ ] Git repo size has decreased
