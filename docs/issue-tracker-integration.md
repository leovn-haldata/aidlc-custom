# Issue Tracker Integration

## Overview

AIDLC uses `<change-id>` as the primary identifier throughout the workflow. This guide defines how `<change-id>` maps to your issue tracker.

## Change ID Format

### Convention

```
<change-id> = <action>-<short-description>
```

Examples: `fix-login-timeout`, `add-csv-export`, `migrate-auth-to-jwt`

### Mapping to Issue Trackers

| Tracker | Change ID Strategy | Example |
|---------|-------------------|---------|
| **GitHub Issues** | Use issue number as suffix: `add-csv-export-42` | Issue #42 → `add-csv-export-42` |
| **Jira** | Use Jira key directly: `PRJ-123` | Jira PRJ-123 → `PRJ-123` |
| **Linear** | Use Linear ID: `LIN-456` | Linear LIN-456 → `LIN-456` |
| **Azure DevOps** | Use work item ID: `AB-789` | Work item #789 → `AB-789` |
| **No tracker** | Use descriptive name: `add-csv-export` | No external reference |

Choose ONE strategy per project and document it in `.aidlc/config.md`.

## Linking Workflow

### At Intake (`/aidlc-intake`)

```markdown
# Intake Note: add-csv-export-42

## Metadata
- **Change ID**: add-csv-export-42
- **Issue**: #42 (https://github.com/org/repo/issues/42)
- **Spec Mode**: feature-spec
```

### In Commit Messages

```
feat(csv-export): add background job for CSV generation

Change-Id: add-csv-export-42
Closes: #42
```

### In PR Descriptions

```markdown
## Change Reference
- **Change ID**: add-csv-export-42
- **Issue**: Closes #42
- **Spec**: `aidlc-docs/changes/active/add-csv-export-42/`
```

### In Release Notes

```markdown
## [1.3.0] - 2026-04-15

### Added
- CSV export for survey results ([add-csv-export-42](aidlc-docs/changes/released/2026-Q2/add-csv-export-42/)) -- Closes #42
```

## Platform-Specific Setup

### GitHub Issues

**Branch auto-link**: GitHub automatically links branches/PRs that reference `#N`.

```markdown
<!-- In PR template -->
Closes #<issue-number>
```

**Labels mapping**:

| Spec Mode | GitHub Label |
|-----------|-------------|
| micro | `micro` (gray) |
| light-spec | `light-spec` (blue) |
| feature-spec | `feature-spec` (green) |
| full-spec | `full-spec` (purple) |
| hotfix | `hotfix` (red) |

### Jira

**Branch naming**: Include Jira key in branch name for auto-linking:
```
feature/PRJ-123-add-csv-export
```

**Smart commits**: Jira recognizes commit messages with ticket keys:
```
PRJ-123 feat(csv): add CSV export endpoint

Change-Id: PRJ-123
```

**Transition triggers** (optional):
| AIDLC Gate | Jira Transition |
|------------|----------------|
| `/aidlc-confirm` passed | Move to "In Progress" |
| `/aidlc-review` accept | Move to "In Review" |
| `/aidlc-release` | Move to "Done" |

### Linear

**Branch naming**: Linear auto-links when branch contains the issue ID:
```
feature/LIN-456-add-csv-export
```

**PR body**: Include Linear issue link:
```markdown
Fixes LIN-456
```

## Sprint Backlog Integration

Sprint files (`aidlc-docs/sprints/`) should link to both the change directory and the issue:

```markdown
## Changes in this Sprint
| Change | Issue | Owner | Mode | Status |
|--------|-------|-------|------|--------|
| [add-csv-export-42](../changes/active/add-csv-export-42/) | [#42](https://github.com/org/repo/issues/42) | @dev-a | feature-spec | IN_PROGRESS |
```

## Rules

- Choose ONE change-id strategy per project (configured in `.aidlc/config.md`)
- Every change MUST reference its issue tracker link in `intake-note.md`
- Every PR MUST reference the change-id AND issue tracker link
- Do not create orphan changes (changes without an issue) unless micro mode
- Micro mode may skip issue creation for truly trivial fixes (team policy)
