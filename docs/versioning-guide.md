# Versioning & CHANGELOG Guide

## Semantic Versioning (SemVer)

Use [Semantic Versioning 2.0.0](https://semver.org/):

```
MAJOR.MINOR.PATCH
  │      │     └── Bug fixes, security patches (backward compatible)
  │      └──────── New features (backward compatible)
  └─────────────── Breaking changes (not backward compatible)
```

### Version Bump Rules

| Change Type | Version Bump | Spec Mode (typical) |
|-------------|-------------|-------------------|
| Typo, config fix | PATCH (1.2.3 → 1.2.4) | micro |
| Bug fix | PATCH (1.2.3 → 1.2.4) | micro / light-spec |
| New feature (backward compatible) | MINOR (1.2.3 → 1.3.0) | feature-spec |
| Breaking API / DB change | MAJOR (1.2.3 → 2.0.0) | full-spec |
| Security hotfix | PATCH (1.2.3 → 1.2.4) | hotfix |

### Pre-Release Versions

For staging / pre-release deployments:

```
1.3.0-rc.1    (release candidate 1)
1.3.0-rc.2    (release candidate 2 -- after UAT fixes)
1.3.0         (final release)
```

## Git Tagging Strategy

```bash
# After merging to master (production deploy)
git tag -a v1.3.0 -m "Release v1.3.0: Add CSV export feature"
git push origin v1.3.0

# For pre-release
git tag -a v1.3.0-rc.1 -m "Release candidate 1 for v1.3.0"
```

### Tag Naming Convention

| Pattern | Usage |
|---------|-------|
| `v1.2.3` | Production release |
| `v1.2.3-rc.N` | Release candidate (staging) |
| `v1.2.3-hotfix.N` | Hotfix release |

## CHANGELOG

Maintain a `CHANGELOG.md` in the project root. This is the **product-level** changelog (aggregates individual change release notes).

### CHANGELOG Format

Follow [Keep a Changelog](https://keepachangelog.com/):

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- <new feature description> ([CHG-xxx](aidlc-docs/changes/released/...))

### Changed
- <change description> ([CHG-xxx](...))

### Fixed
- <bug fix description> ([CHG-xxx](...))

## [1.3.0] - 2026-04-15

### Added
- CSV export for survey analysis results ([CHG-020](aidlc-docs/changes/released/2026-Q2/add-csv-export/))

### Fixed
- Login timeout on slow connections ([CHG-016](aidlc-docs/changes/released/2026-Q2/fix-login-timeout/))

## [1.2.0] - 2026-03-01

### Added
- Multi-tenant AI pipeline ([CHG-010](aidlc-docs/changes/released/2026-Q1/add-ai-pipeline/))

### Changed
- Upgraded Qwen model to v3 ([CHG-011](aidlc-docs/changes/released/2026-Q1/upgrade-qwen-v3/))
```

### CHANGELOG Categories

| Category | When to Use |
|----------|------------|
| **Added** | New features |
| **Changed** | Changes to existing functionality |
| **Deprecated** | Features that will be removed in future |
| **Removed** | Features removed |
| **Fixed** | Bug fixes |
| **Security** | Security patches |

### Linking to AIDLC Artifacts

Each CHANGELOG entry should link to its change's released summary:

```markdown
- CSV export for survey results ([CHG-020](aidlc-docs/changes/released/2026-Q2/add-csv-export/))
```

This creates traceability: CHANGELOG → released summary → (if needed) archived full spec.

## Integration with AIDLC Commands

### When to Update CHANGELOG

| Command | Action |
|---------|--------|
| `/aidlc-release` | Add entry to `[Unreleased]` section |
| Production deploy | Move `[Unreleased]` entries to new version header |
| `/aidlc-hotfix` | Add entry to `[Unreleased]` under `Security` or `Fixed` |

### Release Workflow

```
1. Accumulate changes in [Unreleased] section
2. When deploying to production:
   a. Replace [Unreleased] with [X.Y.Z] - YYYY-MM-DD
   b. Add empty [Unreleased] section at top
   c. Create git tag vX.Y.Z
   d. Deploy
```

## Commit Message Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer: Change-Id: CHG-xxx]
```

### Types

| Type | Description | Version Bump |
|------|-------------|-------------|
| `fix` | Bug fix | PATCH |
| `feat` | New feature | MINOR |
| `feat!` or `BREAKING CHANGE` | Breaking change | MAJOR |
| `docs` | Documentation only | None |
| `style` | Formatting, no code change | None |
| `refactor` | Code restructuring, no behavior change | None |
| `perf` | Performance improvement | PATCH |
| `test` | Adding/fixing tests | None |
| `chore` | Build, CI, tooling changes | None |
| `hotfix` | Production hotfix | PATCH |

### Examples

```
feat(csv-export): add CSV download for survey results

Implements background job processing with GCS storage.
Pre-signed URLs used for secure download.

Change-Id: CHG-020
```

```
fix(auth): increase login timeout to 30s for slow connections

Change-Id: CHG-016
```

```
hotfix(pii): fix phone number masking in free text

Phone numbers in XX-XXXX-XXXX format were not being masked
by the GiNZA pipeline.

Change-Id: hotfix-042
```
